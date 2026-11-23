# Architecture

**Design baseline:** 2026-11-23. This document defines responsibilities and boundaries; it does not assume the two operating systems are on the same machine, same LAN, in a VM, or connected through WSL.

## 1. Two environments

| Environment | Primary role | Tools that belong here |
|---|---|---|
| Windows 10 x64 | Windows desktop target inspection, GUI debuggers, Windows-native runtime | x64dbg, WinDbg if separately required, Ghidra GUI, Frida, ADB, JADX/Apktool, ILSpy, Wireshark, mitmproxy, Node, REA if DSH is installed here |
| Ubuntu 22.04 x64 | Linux-native inspection, headless automation, server-side MCP processes, repeatable CLI workflows | REA if DSH is installed here, Ghidra 12.2+, GDB, Rizin, Frida, angr, Wireshark/tshark, mitmproxy, LASIK, ReClarity coverage with `bwrap`, `ghidra-rpc` |

The split is intentional. A Windows-only process does not gain access to Linux executables just because Ubuntu is also installed. A Linux process does not gain access to Windows process memory. Cross-environment analysis requires an explicitly configured mechanism: transfer evidence files, expose a network service, use SSH, or run an agent client on both environments. None of those links is assumed by this project.

### DSH placement rule

- Install DSH and register REA on each operating environment only if DSH itself is actually installed and supported there.
- The official REA setup command writes the local DSH user's Cordis home patch. It does not create a remote execution bridge.
- If DSH is installed only on Windows, it can use Windows-local REA and tools. Use Ubuntu separately for Linux analysis, then transfer case evidence/results using a chosen secure channel.
- If DSH is installed only on Ubuntu, the inverse applies.
- If there are separate DSH installations, configure and validate each one independently. Do not copy one host's absolute paths into the other host's Cordis patch.

## 2. Logical layers

```text
                 DSH + DeepSeek
                        |
            REA MCP + REA workflow skill
                        |
           +------------+-------------+
           |            |             |
       Static         Dynamic       Network
       analysis       analysis      analysis
           |            |             |
     Ghidra/Rizin   Frida/GDB/      Wireshark/
     JADX/ILSpy/    x64dbg          mitmproxy
     CFR/Apktool
           |            |             |
           +------------+-------------+
                        |
            Evidence and reconstruction
                        |
       tests + traces + decompiled source + notes
                        |
            Understand Anything (post-RE)
                        |
       knowledge graph + architecture exploration
```

Anti-BLA preprocessing (LASIK and optionally ReClarity) is an **input-processing branch**, not a permanent layer through which every target must pass.

## 3. Responsibilities and overlap

### REA

Primary AI-facing investigation interface. It organizes investigation tasks and calls available analysis providers. It is not a replacement for the underlying disassembler, runtime debugger or packet capture engine. REA only exposes capabilities supported by the installed provider and host; check `rea doctor` and the runtime tool availability rather than assuming every provider is active.

### Ghidra

Primary native-code static analysis engine. Use for program import, disassembly, decompilation, function navigation, references, types and program analysis. Preferred host for REA's current Ghidra provider is Ubuntu x64. The current REA documentation describes Windows x64 provider status as experimental P0. Windows Ghidra GUI can still be used independently.

As checked on 2026-11-23, Ghidra 12.2 is the current release line and requires 64-bit JDK 25. REA's current Ghidra provider accepts Ghidra 12.2 or later. REA also documents known flow-recovery limitations in Ghidra 12.2.0; see [Validation](VALIDATION-AND-BENCHMARK.md).

### Rizin

Independent second opinion for binary headers, strings, functions, disassembly, references and scripting. Do not assume its results match Ghidra's decompiler. Use disagreement as a verification signal.

### Dynamic analysis

- Frida: attach or spawn and observe functions, arguments, returns, modules and runtime state where platform permissions permit.
- x64dbg: Windows user-mode debugging.
- GDB: Linux user-mode debugging.

### Network

- Wireshark/tshark: packet and transport-layer evidence.
- mitmproxy: HTTP(S), WebSocket and application-proxy evidence when client configuration permits interception.
- Scapy: packet construction and protocol experiments for authorized lab traffic.

Packet capture is not synonymous with TLS decryption. TLS 1.3, certificate pinning, mutual TLS, custom transports and app-level encryption can limit what a proxy can read.

### Platform-specific source recovery

- Android: JADX for DEX/Java/Kotlin-oriented decompilation; Apktool for resources and smali; ADB for device/emulator communication; Frida for runtime inspection.
- .NET: ILSpy/ilspycmd for managed assemblies.
- Java: CFR for JVM bytecode; JADX is also used for Android DEX.
- Electron: inspect ASAR, JavaScript bundles, source maps when present, Chromium DevTools, Playwright and IPC boundaries.
- iOS: inspect IPA contents and Mach-O with static tooling. Full Apple development/debugging workflow is outside the two-host capability baseline.

### Understand Anything

Post-RE comprehension stage. It analyzes a source tree or documentation set and produces a knowledge graph. It does not analyze the original executable as a binary and does not establish truth of the reverse-engineered behavior. The graph should point back to evidence and test results.

## 4. Case folder model

Use a separate folder per authorized target. The example paths are conventions, not facts about your actual disks.

```text
re-lab/
└── cases/
    └── CASE-ID/
        ├── intake/
        │   └── originals/          # immutable original files
        ├── manifest.yaml           # hashes, provenance, versions, notes
        ├── evidence/
        │   ├── static/             # Ghidra/Rizin/JADX/ILSpy exports
        │   ├── dynamic/            # Frida/debugger traces
        │   └── network/            # PCAP/HAR/proxy exports
        ├── decompilation/          # generated decompiler output
        ├── reconstruction/
        │   ├── src/                # reconstructed/clean source tree
        │   ├── tests/              # behavior-equivalence tests
        │   ├── docs/               # evidence-backed architecture notes
        │   └── .ua/                # Understand Anything output, when supported
        └── reports/
```

Never make generated decompiler output look like original source. Preserve the original binary, record its SHA-256, and keep transformations in separate folders.

## 5. Data-flow rules

1. Hash source inputs before processing.
2. Keep originals read-only where feasible.
3. Store each transform's tool version, command line and output hash.
4. Do not merge observations from two operating systems without recording which host produced each one.
5. Keep packet captures and memory dumps out of public repositories unless explicitly reviewed and sanitized. They can contain credentials, tokens, personal data and proprietary traffic.
6. Keep model prompts and analysis transcripts separate from target evidence. A model-generated statement is not a measurement.

## 6. Why this is preferable to REA + ReClarity alone

REA + ReClarity is useful for AI-led investigation and targets deliberately hardened against blind agents. It does not, by itself, provide a complete platform-specific toolchain for Windows debugging, Linux debugging, Android, iOS, .NET/Java and packet-level analysis. RE-LAB adds those engines and an evidence/reconstruction process. It is more components to maintain; that is a real trade-off, not a benchmark claim that every new component improves every task.
