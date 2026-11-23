# DSH integration

**Current-state check:** 2026-11-23.  
**Primary supported integration:** REA MCP.  
**Understand Anything DSH plugin:** upstream issue and PR are open; not a released integration at the checked date.

## 1. How DSH registers external MCP servers

DeepSeek Harness uses Cordis YAML patch layers, not the generic `mcpServers` JSON block used by some other clients. REA now documents first-class DSH setup. Let REA write its own patch so it can calculate absolute runtime/entrypoint paths, preserve unrelated Cordis patches and set its recommended timeout.

Official reference: [REA — DeepSeek Harness client configuration](https://github.com/morluto/rea/blob/main/docs/getting-started/client-configuration.md#deepseek-harness).

## 2. Install REA for the DSH host

Run this on the same operating environment and user account where DSH runs:

```bash
npm install --global rea-agents@latest
rea --version
rea setup --client dsh
rea doctor --client dsh
```

Windows PowerShell uses the same commands after Node.js/npm are installed:

```powershell
npm install --global rea-agents@latest
rea --version
rea setup --client dsh
rea doctor --client dsh
```

Use a trusted working directory outside the target sample and outside an untrusted project. Review the exact plan before allowing changes. Restart DSH after setup.

### What the setup changes

The current REA documentation says it writes or updates the REA entry in:

- `$DSH_HOME/cordis.patch.yml`, when `DSH_HOME` is set; or
- `~/.dsh/cordis.patch.yml` by default.

The generated registration uses:

- package/client name `@deepseek-ai/dsh-mcp-client`;
- `serverName: rea`;
- `transport: stdio`;
- an absolute Node runtime path;
- an absolute REA `scripts/rea.mjs` path followed by `mcp`;
- `toolCallTimeoutMs: 450000`.

Do not replace these with an example `mcpServers` block. The 450-second timeout is intentional: REA documents that native provider startup, including a cold Ghidra start, can exceed DSH's default 60-second tool timeout.

## 3. Install and verify the REA skill

The REA skill is separate from the MCP registration. Review the proposed install destination before applying it:

```bash
rea setup --skill --dry-run
rea setup --skill --yes
rea doctor --skill --json
```

PowerShell equivalents are the same command names. DSH discovers the shared agent skills directory, normally `~/.agents/skills`, or `$DSH_AGENTS_HOME/skills` when configured. The skill is loaded through DSH's skill mechanism; it is not a separate MCP server.

## 4. DSH on both hosts versus one host

No installation fact has been supplied that proves DSH runs on Windows and Ubuntu. Use the following rule:

| Actual DSH deployment | Required setup |
|---|---|
| DSH runs only on Windows | Register Windows-local REA. Ubuntu analysis remains a separate workflow until a secure remote mechanism is configured. |
| DSH runs only on Ubuntu | Register Ubuntu-local REA. Windows target analysis remains separate until a secure remote mechanism is configured. |
| DSH runs on both | Register REA independently on each host; never reuse another host's absolute executable paths. |
| DSH runs inside a VM/WSL/container | Register against that DSH process's actual filesystem and environment. Host-installed binaries are not automatically visible inside it. |

Do not assume that `localhost` on Windows and `localhost` on Ubuntu identify the same service. Do not put an unauthenticated remote MCP endpoint on a public interface.

## 5. Specialist MCP servers

Keep REA as the only required DSH MCP initially. Add other MCP servers only when they provide a needed stateful workflow and have passed a compatibility test with your exact DSH build.

Candidates:

- [Rizin MCP](https://github.com/rizinorg/rizin-mcp): real MCP server for Rizin. Its repository documents standard MCP transport, but this guide does not assert a tested Cordis patch for your specific DSH version. Try in a disposable DSH profile; default tools are mostly read-only and writing/debugging capabilities require options.
- LASIK: has CLI and an MCP setup command for supported coding-agent integrations. This guide uses its CLI under DSH rather than assuming DSH plugin compatibility.
- ghidra-rpc: **not an MCP server**. It is a local CLI/RPC daemon. REA's Ghidra provider is the supported MCP route here.
- Frida: use CLI/scripts first. Do not expose unrestricted process instrumentation as a general-purpose agent tool without command allowlists and a dedicated test account.
- ReClarity: uses skills/scripts and Bun/`bwrap` for coverage; do not claim direct DSH plugin registration unless you test the exact skill discovery path.

## 6. Troubleshooting sequence

1. `node --version` and `npm --version` work in the same environment as DSH.
2. `rea --version` reports the expected installed version.
3. `rea doctor --client dsh` reports the REA entry healthy.
4. Verify the exact `$DSH_HOME` value and patch file path.
5. Fully restart DSH; a running process may have cached its Cordis layers.
6. Check that DSH discovers `mcp__rea__...` tools.
7. Run a small, harmless analysis and verify the tool response.
8. If native analysis fails, run `rea doctor --provider ghidra --json` on the same host. The terminal's environment is not proof of the environment inherited by a different process.
9. Check DSH logs for the absolute Node and REA script paths and process errors.
10. Do not increase timeouts blindly. REA already sets 450,000 ms for DSH; check the actual configuration if a request still times out.

## 7. Version and reproducibility policy

For a first interactive setup, `@latest` is convenient. For a reproducible lab, record `rea --version`, then pin the tested package version in your deployment procedure. Do not assume the version used on Windows and Ubuntu is identical. Update deliberately, review the setup plan, rerun `rea doctor`, and re-run the benchmark tests.
