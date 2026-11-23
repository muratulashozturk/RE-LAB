# Ubuntu 22.04 x64 setup

**Scope:** Linux-native targets, headless analysis, REA/Ghidra provider, packet processing and optional anti-BLA runtime coverage.  
**Checked:** 2026-11-23.

## 1. System packages

Ubuntu 22.04's default Python is not guaranteed to satisfy every current Python tool. Keep system Python untouched and use a managed Python 3.12 environment for Frida/mitmproxy/angr as needed.

```bash
sudo apt update
sudo apt install -y \
  git curl wget ca-certificates unzip zip xz-utils \
  build-essential cmake ninja-build pkg-config \
  gcc g++ make binutils file patchelf \
  gdb strace ltrace \
  tcpdump tshark wireshark \
  jq ripgrep sqlite3 \
  bison flex zlib1g-dev libssl-dev libffi-dev
```

A package may be absent from a configured Ubuntu mirror or require enabling an official repository. Check `apt-cache policy <package>` before assuming availability. Do not replace Ubuntu's system Python to satisfy a tool.

Optional non-root packet capture:

```bash
sudo usermod -aG wireshark "$USER"
```

Log out and back in. Verify that the account is a member of the expected group and that the selected interface is visible. Packet capture permissions depend on the system's Wireshark/tshark packaging and capture configuration.

## 2. Workspace

```bash
mkdir -p "$HOME/re-lab"/{tools,cases,venvs,cache,reports}
```

This is a convention under the current user's home directory. It is not a cross-OS shared folder.

## 3. Node.js supported by REA

REA currently supports Node.js 22.x >=22.23.4, 24.x >=24.22.1, and 26.12.1+. Unsupported releases include Node 23, 25, earlier 26.x and prereleases. Use an exact supported version from [nodejs.org](https://nodejs.org/en/download/) or a version manager; do not select a generic LTS release without checking its version.

One version-manager method is [nvm](https://github.com/nvm-sh/nvm). After installing nvm using its official instructions, install a currently supported Node 24.x patch at or above 24.22.1:

```bash
nvm install 24
nvm use 24
node --version
npm --version
```

If the installed `24` alias does not resolve to >=24.22.1, install the exact patch explicitly. Confirm support in the REA installation guide before changing major versions.

## 4. REA + DSH (only if DSH runs on Ubuntu)

```bash
npm install --global rea-agents@latest
rea --version
rea setup --client dsh
rea doctor --client dsh
```

Review the proposed write. The default DSH home patch is `$HOME/.dsh/cordis.patch.yml`; set `DSH_HOME` only if your real DSH installation uses another home. Restart DSH after setup.

If DSH is not installed/running on Ubuntu, do **not** run this section expecting to connect the Windows DSH process to Ubuntu. Use [DSH integration](DSH-INTEGRATION.md).

Install/refresh the REA workflow skill if needed:

```bash
rea setup --skill --dry-run
# Inspect the proposed destination and changes.
rea setup --skill --yes
rea doctor --skill --json
```

## 5. JDK 25 and Ghidra 12.2

Ghidra 12.2 (October 2026) requires a 64-bit JDK 25. Download a free 64-bit JDK 25 build from [Eclipse Temurin](https://adoptium.net/temurin/releases/) or [Amazon Corretto](https://docs.aws.amazon.com/corretto/latest/corretto-25-ug/downloads-list.html). Do not assume `openjdk-25-jdk` is present in Ubuntu 22.04's default APT repositories.

Download the official Ghidra 12.2 multi-platform release from [NSA Ghidra releases](https://github.com/NationalSecurityAgency/ghidra/releases). Download the release ZIP asset, not the two GitHub "Source code" archives. Extract into a new directory, not over an older version.

Set paths for the current shell (change them to actual extracted names):

```bash
export JAVA_HOME="$HOME/tools/jdk-25"
export PATH="$JAVA_HOME/bin:$PATH"
export GHIDRA_INSTALL_DIR="$HOME/re-lab/tools/ghidra_12.2_PUBLIC"

java --version
 test -x "$GHIDRA_INSTALL_DIR/ghidraRun"
"$GHIDRA_INSTALL_DIR/ghidraRun"
```

Add exports to `~/.bashrc` only after confirming the paths. Do not commit machine-specific paths into this repository.

### REA provider check

```bash
rea doctor --provider ghidra --json
```

The current REA Ghidra provider supports Linux x64/arm64 and requires Ghidra >=12.2 with the JDK declared by that Ghidra installation. Verify the `ghidra` provider reports ready on this exact host. REA does not install Ghidra or Java for you. See the current [REA Ghidra provider guide](https://github.com/morluto/rea/blob/main/docs/getting-started/installation.md#ghidra).

**Known Ghidra 12.2.0 limitation:** REA's live provider documentation records a flow-reference defect for some Clang `-O0` x86-64 switch tables in Ghidra 12.2.0 and an AArch64 switch recovery issue. Check the current REA provider guide and Ghidra issue status before relying on affected control-flow output. Cross-check suspicious functions with disassembly, Rizin and runtime traces. See [Validation](VALIDATION-AND-BENCHMARK.md).

## 6. ghidra-rpc (optional; it is a CLI/RPC daemon, not an MCP server)

This can be useful for a persistent local Ghidra project and scripted CLI access. It is not required when REA's Ghidra provider is already sufficient.

Prerequisites: Ghidra 11+ according to the `ghidra-rpc` installation guide, Python 3.11+, Java required by your Ghidra release (JDK 25 for Ghidra 12.2), and `uv`.

Install `uv` using its [official instructions](https://docs.astral.sh/uv/getting-started/installation/):

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source "$HOME/.local/bin/env" 2>/dev/null || true
uv --version
```

Install ghidra-rpc from its repository:

```bash
git clone https://github.com/cellebrite-labs/ghidra-rpc.git "$HOME/re-lab/tools/ghidra-rpc"
uv tool install "$HOME/re-lab/tools/ghidra-rpc"
```

Set the Ghidra directory and test:

```bash
export GHIDRA_INSTALL_DIR="$HOME/re-lab/tools/ghidra_12.2_PUBLIC"
ghidra-rpc --version
```

Example use (the project and target paths here are examples):

```bash
ghidra-rpc start --project "$HOME/re-lab/cases/CASE-ID/working/ghidra-project.gpr" --headless
export GHIDRA_RPC_PROJECT="$HOME/re-lab/cases/CASE-ID/working/ghidra-project.gpr"
ghidra-rpc load /usr/bin/ls
ghidra-rpc decompile ls main
```

Check current upstream CLI syntax with `ghidra-rpc --help`. On Windows, ghidra-rpc uses an authenticated loopback TCP socket and stores a descriptor under `%LOCALAPPDATA%`; treat that descriptor as a credential. The upstream guide documents this behavior. [ghidra-rpc install guide](https://github.com/cellebrite-labs/ghidra-rpc/blob/main/docs/install.md).

## 7. Rizin / Cutter

Install Rizin from the [official installation guide](https://rizin.re/) or official releases. Do not assume the Ubuntu 22.04 archive contains the current version. Check:

```bash
rizin -v
```

Cutter GUI releases: [Rizinorg/Cutter](https://github.com/rizinorg/cutter/releases). It is optional; Rizin CLI is enough for scripted analysis.

Optional [Rizin MCP](https://github.com/rizinorg/rizin-mcp) exposes Rizin through MCP, but it is not part of the baseline DSH configuration. Validate its exact binary requirements, tool safety settings and DSH Cordis registration in a disposable profile before adding it. Its default tool set is read-oriented; patching/debugging tools require enabling options.

## 8. Frida

Use a managed environment; do not install Python packages into Ubuntu's externally managed system Python.

```bash
uv python install 3.12
uv venv --python 3.12 "$HOME/re-lab/venvs/frida"
source "$HOME/re-lab/venvs/frida/bin/activate"
uv pip install frida-tools
frida --version
frida-ps -a
```

Install a device-side Frida server or gadget only when required and match its version and architecture to the client. Permissions, sandboxing and device state can prevent attachment.

## 9. mitmproxy, Scapy and packet tools

```bash
uv python install 3.12
uv venv --python 3.12 "$HOME/re-lab/venvs/network"
source "$HOME/re-lab/venvs/network/bin/activate"
uv pip install mitmproxy scapy
mitmweb --version
scapy --help
```

`mitmweb` normally listens on `127.0.0.1:8080` for proxy traffic and `127.0.0.1:8081` for its UI; confirm the actual console output. `tshark --version` and `wireshark --version` validate packet tooling.

TLS interception requires the test client to trust the mitmproxy CA and permit proxying. Pinning, mTLS, custom TLS, certificate transparency policies and application-level encryption may prevent useful plaintext capture.

## 10. angr (advanced, optional)

Use a separate environment to avoid dependency conflicts:

```bash
uv venv --python 3.12 "$HOME/re-lab/venvs/angr"
source "$HOME/re-lab/venvs/angr/bin/activate"
uv pip install angr
python -c 'import angr; print(angr.__version__)'
```

angr is for selected symbolic/concolic analysis tasks; it is not a universal automatic decompiler. Start with small, well-understood functions and explicit constraints.

## 11. LASIK and ReClarity

LASIK static preprocessing:

```bash
npm install --global lasik-agents
lasik --help
lasik doctor
lasik scan /absolute/path/to/target
lasik heal /absolute/path/to/target
```

LASIK requires Node.js 20.19 or later and says it never executes the target. It writes a cleaned copy (for example `<input>.lasik`); preserve and hash the original. Its supported-format/coverage figures are project-maintainer-reported and should be validated against your own target.

ReClarity is an optional workflow for BLA-hardening signals and runtime coverage. Upstream requires Bun and, for coverage, `bwrap`. Use it on Ubuntu for compatible Linux/Electron samples; a Windows-only executable cannot be assumed runnable under the Linux coverage stage. Follow the repository's current [setup](https://github.com/i098/ReClarity/blob/main/setup.md) and [usage guide](https://github.com/i098/ReClarity/blob/main/docs/usage.md). Do not apply it to every target by default.

## 12. Understand Anything

Do not treat it as a binary-analysis install. It is a post-RE codebase mapping tool. Upstream has a general Linux installer for several agent platforms, but DSH-specific bundle support remains an open upstream issue/PR as of 2026-11-23. See [Understand Anything](UNDERSTAND-ANYTHING.md) before installing it.

## 13. Verify Ubuntu toolchain

```bash
node --version
npm --version
python3 --version
java --version
rea --version
rea doctor --json
rea doctor --provider ghidra --json
frida --version
rizin -v
gdb --version
tshark --version
mitmweb --version
```

Record output with environment, date and host name in the case report. A command existing on `PATH` does not prove that its analysis provider is configured or works for the target architecture.
