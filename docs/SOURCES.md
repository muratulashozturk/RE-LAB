# Sources and verification notes

**Last checked:** 2026-11-23.  
These are upstream project and documentation links used to build this guide. Tool versions and compatibility can change; re-check before installation or upgrades.

## Core orchestration and DSH

- [REA repository](https://github.com/morluto/rea)
- [REA installation guide](https://github.com/morluto/rea/blob/main/docs/getting-started/installation.md)
- [REA client configuration — DSH](https://github.com/morluto/rea/blob/main/docs/getting-started/client-configuration.md#deepseek-harness)
- [DeepSeek Harness repository](https://github.com/deepseek-ai/deepseek-harness) — verify the exact client/version in use; the user's DSH version was not supplied.

## Native static and dynamic analysis

- [Ghidra releases](https://github.com/NationalSecurityAgency/ghidra/releases)
- [Ghidra 12.2 change history](https://www.ghidradocs.com/12.2_PUBLIC/docs/ChangeHistory.html)
- [Ghidra 12.2 getting started](https://github.com/NationalSecurityAgency/ghidra/blob/Ghidra_12.2_build/GhidraDocs/GettingStarted.md)
- [REA Ghidra provider guide](https://github.com/morluto/rea/blob/main/docs/getting-started/installation.md#ghidra)
- [Ghidra issue #9996](https://github.com/NationalSecurityAgency/ghidra/issues/9996)
- [Ghidra issue #10154](https://github.com/NationalSecurityAgency/ghidra/issues/10154)
- [ghidra-rpc repository](https://github.com/cellebrite-labs/ghidra-rpc)
- [ghidra-rpc installation guide](https://github.com/cellebrite-labs/ghidra-rpc/blob/main/docs/install.md)
- [Rizin](https://github.com/rizinorg/rizin)
- [Cutter](https://github.com/rizinorg/cutter)
- [Rizin MCP](https://github.com/rizinorg/rizin-mcp)
- [Frida installation docs](https://frida.re/docs/installation/)
- [Frida repository](https://github.com/frida/frida)
- [x64dbg](https://github.com/x64dbg/x64dbg)
- [GDB](https://www.sourceware.org/gdb/)
- [angr](https://github.com/angr/angr)

## Platform analysis

- [JADX](https://github.com/skylot/jadx)
- [Apktool](https://github.com/iBotPeaches/Apktool)
- [Android Platform Tools](https://developer.android.com/tools/releases/platform-tools)
- [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF)
- [ILSpy](https://github.com/icsharpcode/ILSpy)
- [CFR](https://github.com/leibnitz27/cfr)
- [Playwright](https://github.com/microsoft/playwright)
- [Electron ASAR archives](https://www.electronjs.org/docs/latest/tutorial/asar-archives)
- [libimobiledevice](https://github.com/libimobiledevice/libimobiledevice)

## Network and identification

- [Wireshark](https://www.wireshark.org/)
- [mitmproxy docs](https://docs.mitmproxy.org/stable/)
- [mitmproxy repository](https://github.com/mitmproxy/mitmproxy)
- [Scapy](https://github.com/secdev/scapy)
- [YARA](https://github.com/VirusTotal/yara)
- [capa](https://github.com/mandiant/capa)

## Anti-BLA and post-RE understanding

- [LASIK](https://github.com/pktq/lasik)
- [ReClarity](https://github.com/i098/ReClarity)
- [ReClarity usage](https://github.com/i098/ReClarity/blob/main/docs/usage.md)
- [Understand Anything](https://github.com/Egonex-AI/Understand-Anything)
- [Understand Anything DSH issue #670](https://github.com/Egonex-AI/Understand-Anything/issues/670)
- [Understand Anything DSH bundle PR #673](https://github.com/Egonex-AI/Understand-Anything/pull/673)

## Specific verified facts used in this documentation

1. REA's DSH setup command is `rea setup --client dsh`; DSH uses Cordis YAML patch layers; default patch path is `~/.dsh/cordis.patch.yml`; setup configures a 450,000 ms REA MCP tool timeout and exposes tool names under `mcp__rea__...`.
2. REA currently documents Node.js support as 22.x >=22.23.4, 24.x >=24.22.1 and 26.12.1+.
3. Ghidra 12.2 was released in October 2026 and requires 64-bit JDK 25. REA currently accepts Ghidra >=12.2 and documents Linux x64/arm64 support plus experimental Windows x64 provider status.
4. REA documents known Ghidra 12.2.0 switch flow-reference issues that require cross-validation.
5. `ghidra-rpc` documentation describes a CLI and daemon. It is not represented here as an MCP server.
6. LASIK says it is static and does not execute the input; its format coverage and token reductions are project-reported, not independently validated by this guide.
7. ReClarity's published 0/3 versus 3/3 comparison is explicitly a single run per configuration using one model and one BLA seed on an Electron-style JS/ASAR target. It is not treated as a general performance result.
8. Understand Anything's graph is saved at `.ua/knowledge-graph.json` by default, and its first full scan can use significant model tokens.
9. Understand Anything's DSH issue #670 and PR #673 were open as of the checked date. The DSH plugin is therefore marked pending, not installed or supported.
10. iOS static analysis is possible on Windows/Linux, but a full Xcode/simulator/signing/debugging workflow requires macOS. Actual device-side Frida capability depends on device state and deployment conditions.

## Verification method

This guide was prepared from the upstream repositories, official docs, release history and issue/PR status linked above. No installation was executed on the user's Windows or Ubuntu machines. All local paths in examples are replaceable conventions. Exact DSH version, machine topology, device architecture/state, available disk/RAM and actual installation directories remain unverified and are intentionally not guessed.
