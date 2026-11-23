# Windows 10 x64 setup

**Scope:** Windows desktop target analysis and the Windows-side toolchain.  
**Checked:** 2026-11-23.  
**Important:** Commands below install software only. They do not prove compatibility with every target, DSH build, Windows policy or CPU feature.

## 1. Base prerequisites

Install current supported x64 releases from official sources:

- [Node.js downloads](https://nodejs.org/en/download/)
- [Git for Windows](https://git-scm.com/download/win)
- [Python 3.12 downloads](https://www.python.org/downloads/windows/)
- [PowerShell documentation](https://learn.microsoft.com/powershell/)

REA currently documents Node.js 22.x >=22.23.4, 24.x >=24.22.1, or 26.12.1+. It does not support Node.js 23, 25, earlier 26.x, or prereleases. Select a supported release and verify the exact version before installation. [REA install documentation](https://github.com/morluto/rea/blob/main/docs/getting-started/installation.md#package-runners-and-global-installation).

Check after installing:

```powershell
node --version
npm --version
python --version
git --version
```

## 2. Create a workspace

This path is a proposed convention. Change it if the drive or free space differs on your machine.

```powershell
New-Item -ItemType Directory -Force -Path `
  'D:\RE-LAB\tools', `
  'D:\RE-LAB\cases', `
  'D:\RE-LAB\venvs', `
  'D:\RE-LAB\cache', `
  'D:\RE-LAB\reports' | Out-Null
```

Do not store sensitive captures or proprietary samples in a public Git repository.

## 3. Install REA and register it with DSH

Only perform this section on this Windows environment if DSH is installed and runs under the same Windows user account.

```powershell
npm install --global rea-agents@latest
rea --version
rea setup --client dsh
rea doctor --client dsh
```

Read the setup plan and approve the actual DSH configuration change when it shows the intended file. REA's DSH integration uses `$DSH_HOME\cordis.patch.yml`, defaulting to `$HOME\.dsh\cordis.patch.yml`; it is not a conventional `mcpServers` JSON block. The setup uses absolute Node and REA script paths and a 450,000 ms tool timeout. Restart DSH after setup.

Install the REA workflow skill separately if setup did not install it or you need to refresh it:

```powershell
rea setup --skill --dry-run
# Inspect the target path and proposed changes first.
rea setup --skill --yes
rea doctor --skill --json
```

Do not paste a hand-written generic MCP config into DSH. See [DSH integration](DSH-INTEGRATION.md).

## 4. Ghidra on Windows

Ghidra 12.2 requires 64-bit JDK 25. Download the official multi-platform Ghidra release archive from [Ghidra releases](https://github.com/NationalSecurityAgency/ghidra/releases) and a 64-bit JDK 25 from [Eclipse Temurin](https://adoptium.net/temurin/releases/).

1. Install JDK 25.
2. Extract the Ghidra release archive to a stable path; do not extract over an older Ghidra installation.
3. Set environment variables for the current PowerShell session, using your actual paths:

```powershell
$env:JAVA_HOME = 'C:\Path\To\jdk-25'
$env:GHIDRA_INSTALL_DIR = 'D:\RE-LAB\tools\ghidra_12.2_PUBLIC'
& "$env:GHIDRA_INSTALL_DIR\ghidraRun.bat"
```

The example folder name is illustrative; use the actual extracted directory name. Verify Java and Ghidra:

```powershell
java --version
Test-Path "$env:GHIDRA_INSTALL_DIR\ghidraRun.bat"
```

**Provider limitation:** REA's current docs list Ghidra on Windows x64 as experimental P0. For an REA-managed Ghidra provider, use Ubuntu as the preferred host. Windows Ghidra GUI remains useful independently. Do not claim that a successful GUI launch proves REA's Windows provider works.

## 5. Windows debugger

Download x64dbg only from its official [GitHub releases](https://github.com/x64dbg/x64dbg/releases). Extract to a stable directory. Use the 64-bit debugger for x64 targets and the 32-bit debugger for 32-bit processes. Record the exact release in the case manifest.

For kernel debugging or advanced Windows internals, WinDbg is a separate tool with separate installation and symbol configuration; it is not part of the minimal baseline here.

## 6. Frida

Use an isolated Python environment:

```powershell
py -3.12 -m venv D:\RE-LAB\venvs\frida
D:\RE-LAB\venvs\frida\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install frida-tools
frida --version
frida-ps --local
```

Official instructions: [Frida installation](https://frida.re/docs/installation/). Device-side Frida server/gadget must match the client version and target architecture. Process access may require elevated permissions. Do not infer that an installed CLI can attach to protected processes.

## 7. Network tools

Wireshark: install from [wireshark.org](https://www.wireshark.org/download.html). Confirm whether Npcap is installed and which capture interfaces Windows exposes.

mitmproxy requires Python 3.12 or later in its current project metadata. Use the Python 3.12 environment above or a separate one:

```powershell
py -3.12 -m venv D:\RE-LAB\venvs\mitmproxy
D:\RE-LAB\venvs\mitmproxy\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install mitmproxy
mitmweb --version
mitmweb
```

Default local proxy endpoint is normally `127.0.0.1:8080`, with web UI on `127.0.0.1:8081`. Confirm the actual startup output. Install the mitmproxy CA only in an isolated test profile/device where you control trust settings. Certificate pinning and app-level encryption can prevent interception; do not bypass protections in third-party apps without authorization.

## 8. Android tooling

Install Android SDK Platform Tools from [Google's official page](https://developer.android.com/tools/releases/platform-tools). Verify:

```powershell
adb version
adb devices -l
```

Install JADX and Apktool from their official release pages:

- [JADX releases](https://github.com/skylot/jadx/releases)
- [Apktool releases](https://github.com/iBotPeaches/Apktool/releases)

After installing them and ensuring their executables are on `PATH`:

```powershell
jadx --version
apktool --version

New-Item -ItemType Directory -Force -Path 'D:\RE-LAB\cases\CASE-ID\decompilation\jadx' | Out-Null
jadx -d 'D:\RE-LAB\cases\CASE-ID\decompilation\jadx' 'D:\RE-LAB\cases\CASE-ID\intake\originals\sample.apk'
```

Replace `CASE-ID` and the APK path. Use only a test device or emulator you control. ADB availability does not imply Frida injection support on a locked, non-debuggable device.

## 9. .NET and Java

ILSpy command-line tool:

```powershell
dotnet tool install --global ilspycmd
ilspycmd --help
```

If you already have `ilspycmd`, update only after reviewing the version change. GUI: [ILSpy releases](https://github.com/icsharpcode/ILSpy/releases).

CFR is a Java decompiler. Download the JAR from [official CFR releases](https://github.com/leibnitz27/cfr/releases); verify its checksum when the release publishes one. Example:

```powershell
java -jar 'D:\RE-LAB\tools\cfr.jar' 'D:\RE-LAB\cases\CASE-ID\intake\originals\sample.jar' --outputdir 'D:\RE-LAB\cases\CASE-ID\decompilation\cfr'
```

Use the correct command syntax from the CFR version you downloaded.

## 10. Electron and web

Install Playwright in a project-specific Node workspace rather than globally:

```powershell
New-Item -ItemType Directory -Force -Path 'D:\RE-LAB\tools\playwright-work' | Out-Null
Set-Location 'D:\RE-LAB\tools\playwright-work'
npm init -y
npm install --save-dev playwright
npx playwright --version
```

Install browser binaries only if needed:

```powershell
npx playwright install chromium
```

For Electron, inspect ASAR archives using the current [Electron ASAR documentation](https://www.electronjs.org/docs/latest/tutorial/asar-archives). If installing `@electron/asar`, verify its current package and license before adding it to the approved list; do not rely on an old global package.

## 11. Verification

Run the commands in [Validation and benchmark](VALIDATION-AND-BENCHMARK.md). Record actual versions and command output. Do not mark a provider as available until its specific test passes.
