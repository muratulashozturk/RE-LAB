# RE-LAB: Open-Source AI-Assisted Reverse-Engineering Laboratory

**Documentation baseline:** 2026-11-23  
**Target operating environments:** Windows 10 x64 and Ubuntu 22.04 x64  
**AI client:** DeepSeek Harness (DSH) with DeepSeek  
**License policy:** free/open-source software only; each dependency and model/service is audited separately.

RE-LAB is a documented two-environment workflow for investigating software that you are authorized to analyze. It combines REA for AI-orchestrated investigation with specialized static, dynamic, platform and network analysis tools. It adds **Understand Anything after reverse engineering**, to build a navigable knowledge graph from recovered/reconstructed source code and documentation.

This repository is an installation and operating guide. It does not contain the third-party tools themselves, does not claim that every tool has been installed on your machines, and does not infer your DSH version, machine resources, device state, filesystem paths or Windows/Ubuntu network topology.

## Decision summary

| Area | Selected component | Role | Status |
|---|---|---|---|
| AI investigation | [REA](https://github.com/morluto/rea) | Primary RE MCP and workflow | Primary |
| Native static analysis | [Ghidra](https://github.com/NationalSecurityAgency/ghidra) | Disassembly, decompilation, xrefs, data types | Primary |
| Native CLI analysis | [Rizin](https://github.com/rizinorg/rizin) | Scriptable second opinion | Recommended |
| Native runtime instrumentation | [Frida](https://frida.re/) | Runtime observation and hooks | Primary |
| Windows debugger | [x64dbg](https://github.com/x64dbg/x64dbg) | Windows user-mode debugging | Primary for Windows targets |
| Linux debugger | GDB | Linux user-mode debugging | Primary for Linux targets |
| Network | [Wireshark](https://www.wireshark.org/), [mitmproxy](https://mitmproxy.org/) | Packet and HTTP(S) analysis | Primary |
| Electron / web | Node.js, [Playwright](https://playwright.dev/), Chromium DevTools, ASAR tooling | Bundle inspection and behavior testing | Primary |
| Android | JADX, Apktool, ADB, Frida | APK/DEX/resources/runtime | Primary |
| iOS | Ghidra, archive/Mach-O inspection, libimobiledevice/Frida where supported | Static analysis and limited device workflows | Partial; see limitations |
| .NET | [ILSpy](https://github.com/icsharpcode/ILSpy) / `ilspycmd` | Assembly inspection and decompilation | Primary |
| Java | [CFR](https://github.com/leibnitz27/cfr) | JVM bytecode decompilation | Primary |
| Symbolic execution | [angr](https://github.com/angr/angr) | Selected path/constraint analysis | Optional advanced |
| Anti-BLA preprocessing | [LASIK](https://github.com/pktq/lasik) | Static cleanup of detected anti-agent hardening | Optional |
| Anti-BLA runtime evidence | [ReClarity](https://github.com/i098/ReClarity) | Additional scanning, sandboxed coverage and budget workflow | Optional; use on Ubuntu for `bwrap` coverage |
| Post-RE understanding | [Understand Anything](https://github.com/Egonex-AI/Understand-Anything) | Knowledge graph over recovered/reconstructed code | Required workflow stage; **DSH plugin support pending** |

## Important realities / non-goals

1. **No imaginary cross-host integration.** A Windows DSH process can launch only tools that are reachable from that Windows process. An Ubuntu installation is not automatically visible to it. See [Architecture](docs/ARCHITECTURE.md) and [DSH integration](docs/DSH-INTEGRATION.md).
2. **REA has an official DSH setup path.** Use `rea setup --client dsh`; do not hand-write an unrelated `mcpServers` JSON object. DSH uses Cordis YAML patches. REA's current documentation says the setup writes `~/.dsh/cordis.patch.yml` (or `$DSH_HOME/cordis.patch.yml`) and sets a 450,000 ms tool timeout. [Official REA client configuration](https://github.com/morluto/rea/blob/main/docs/getting-started/client-configuration.md#deepseek-harness).
3. **Understand Anything is downstream of RE.** It maps source code, functions, classes and dependencies. It is not a decompiler, unpacker, debugger or binary reverse-engineering engine.
4. **Understand Anything does not currently have a merged DSH integration.** As checked on 2026-11-23, upstream issue [#670](https://github.com/Egonex-AI/Understand-Anything/issues/670) and DSH bundle PR [#673](https://github.com/Egonex-AI/Understand-Anything/pull/673) are open. Do not represent the PR as a released, supported DSH plugin. The workflow is included, but its DSH command UX is blocked pending an upstream merge/release or a separately reviewed local port.
5. **iOS cannot be fully covered by Windows 10 + Ubuntu 22.04 alone.** Static IPA/Mach-O inspection is possible. Full Apple-supported simulator, signing, Xcode debugging and some device workflows require macOS. Device instrumentation also depends on device security state and deployment permissions.
6. **Tool license is not model/API price.** Most selected software is FOSS, but DeepSeek API usage may be billable and subject to the model/provider terms. Review those separately. A FOSS client does not make a hosted model free.
7. **No tool guarantees semantic correctness.** Treat decompiler output, AI explanations, protocol hypotheses and generated knowledge graphs as hypotheses until corroborated by independent evidence and tests.

## Quick navigation

- [Architecture and responsibilities](docs/ARCHITECTURE.md)
- [Windows 10 installation](docs/WINDOWS-10.md)
- [Ubuntu 22.04 installation](docs/UBUNTU-22.04.md)
- [DSH + REA integration](docs/DSH-INTEGRATION.md)
- [Operating procedure and post-RE flow](docs/RE-WORKFLOW.md)
- [Understand Anything](docs/UNDERSTAND-ANYTHING.md)
- [Licenses, scope and exclusions](docs/LICENSES-AND-SCOPE.md)
- [Validation and benchmark](docs/VALIDATION-AND-BENCHMARK.md)
- [Sources and verification notes](docs/SOURCES.md)

## Recommended implementation order

1. Install and verify REA + DSH on the host where DSH actually runs.
2. Install Ghidra and JDK 25 on Ubuntu; use that as the preferred REA/Ghidra provider environment. Ghidra on Windows is useful as a GUI, but REA documents its Windows x64 Ghidra provider as experimental P0.
3. Install Frida and the platform-specific debuggers on their respective hosts.
4. Install Wireshark and mitmproxy; test with a controlled application you own or are authorized to inspect.
5. Add Android and .NET/Java toolchains.
6. Add LASIK/ReClarity only when the target or observed behavior gives a reason to use anti-BLA handling.
7. Complete RE, export evidence and create a reconstructed source/documentation tree.
8. Run Understand Anything on that reconstructed tree after a tested DSH integration is available, or use one of its currently supported agent platforms outside DSH.

## State of this documentation

The installation commands are separated from optional integrations and known limitations. Commands that depend on a missing local fact are conditional. Update this documentation after verifying exact versions and paths on both systems; record successful test output in `docs/VALIDATION-AND-BENCHMARK.md` rather than marking an untested command as working.
