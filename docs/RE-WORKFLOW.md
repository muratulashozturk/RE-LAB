# Reverse-engineering operating procedure

This is the repeatable per-target workflow. Use it for software you own or are explicitly authorized to inspect.

## 1. Intake and provenance

Create a case ID and copy the sample into a controlled intake directory. Keep the original separate from all extracted, patched, healed or decompiled material.

Example manifest:

```yaml
case_id: CASE-ID
created_utc: "YYYY-MM-DDTHH:MM:SSZ"
authorization: "Describe the ownership/authorization basis"
target:
  filename: "sample.exe"
  sha256: "REPLACE_WITH_SHA256"
  file_size_bytes: 0
  claimed_platform: "unknown"
  observed_format: "unknown"
environments:
  windows:
    os: "Windows 10 x64"
    hostname: "RECORD_ACTUAL_HOST"
  ubuntu:
    os: "Ubuntu 22.04 x64"
    hostname: "RECORD_ACTUAL_HOST"
tools: {}
notes: []
```

Fill in measured values only. Do not enter guessed file formats, versions, CPU architecture or origin.

Hash the original:

```powershell
Get-FileHash -Algorithm SHA256 -LiteralPath 'PATH_TO_ORIGINAL'
```

```bash
sha256sum 'PATH_TO_ORIGINAL'
```

## 2. Identify the target before selecting engines

Record at least:

- file type and architecture;
- operating system/ABI if known;
- signatures, imports, exports and sections;
- whether the input is an archive/container;
- known symbols/debug information;
- runtime dependencies;
- provenance and hash.

Do not infer target identity from filename or icon alone.

## 3. Static analysis pass

Use the engine suited to the input:

| Input | First-line tooling |
|---|---|
| PE / Windows EXE/DLL | Ghidra, Rizin; PE inspection; x64dbg later if runtime questions remain |
| ELF / Linux executable or SO | Ghidra, Rizin, `readelf`, `objdump`; GDB/Frida later |
| Electron | Archive inspection, JS/TS bundles, source maps if present, REA, LASIK scan if relevant |
| APK/DEX | JADX + Apktool; inspect manifest/resources/smali; REA and Frida as needed |
| IPA/Mach-O | Unzip/inspect metadata and Mach-O in Ghidra; note that full iOS runtime workflow needs additional Apple/device conditions |
| .NET assembly | ILSpy/ilspycmd; Ghidra only when native/unmanaged code is material |
| JAR/class | CFR; inspect resources and metadata; use runtime tracing if needed |
| PCAP | Wireshark/tshark; mitmproxy/HAR only when traffic was generated through an authorized test setup |

Save tool output and command lines under `evidence/static/` or `evidence/network/`.

## 4. Anti-BLA decision

Do not automatically run anti-BLA tools on every binary.

1. Run `lasik scan <target>` when the input is a format LASIK supports or there are signs of decoy/prompt-injection hardening.
2. If LASIK reports protection, run `lasik heal <target>` to generate a separate working copy.
3. Preserve the original and both hashes.
4. Compare unmodified and healed artifacts; check that the changes are explained and behavior-preserving claims are tested.
5. Use ReClarity when its coverage checks add value and the target can run inside the supported Ubuntu `bwrap` environment. Keep runtime outputs as evidence; do not treat a function as dead merely because one test did not execute it.

LASIK and ReClarity are not interchangeable: LASIK is static; ReClarity's published workflow adds sandboxed runtime coverage and budget handling. See [LASIK](https://github.com/pktq/lasik) and [ReClarity](https://github.com/i098/ReClarity).

## 5. Dynamic analysis pass

Use dynamic analysis only in a disposable, isolated environment suitable for the target OS and architecture.

- Frida: attach/spawn only where process permissions and target protections permit; log hook location, version, arguments, return values and limitations.
- x64dbg: Windows user-mode debugging.
- GDB: Linux debugging; record symbols, breakpoints and commands.
- Playwright/DevTools: reproducible web/Electron interactions; preserve scripts and browser versions.
- ADB/Frida: Android emulator or test device you control.

Do not run unknown binaries on the analyst's everyday desktop. Use VM snapshots and restricted networking; disable shared clipboard/folders where they could expose credentials. Analysis can still leak data through intentionally enabled networking, so record the network policy.

## 6. Network investigation

Capture at the correct layer:

- Wireshark/tshark for packets and transport behavior;
- mitmproxy for proxy-visible HTTP(S)/WebSocket flows;
- application runtime hooks for plaintext immediately before encryption or after decryption when permitted;
- application-level logs only when independently tied to the target run.

Record interface, proxy settings, time zone, clock source, capture filter and whether TLS was decrypted. A PCAP containing only TLS ciphertext does not establish HTTP semantics. Do not label a guessed protocol field as confirmed.

## 7. Reconstruction

Separate recovered facts from interpretations.

- `evidence/`: original observations and generated tool outputs.
- `decompilation/`: raw decompiler exports, labelled as generated.
- `reconstruction/src/`: source or behavior reimplementation produced by the analyst/agent.
- `reconstruction/tests/`: tests comparing reconstructed and original behavior.
- `reconstruction/docs/`: architecture notes and evidence links.

For each reconstructed function, record:

- original module/file and address or metadata token;
- calling convention/signature where known;
- inputs and outputs;
- control-flow assumptions;
- evidence supporting each behavior claim;
- known unknowns;
- regression tests and result.

Do not call pseudocode “original source code”.

## 8. Post-RE: Understand Anything

After a source tree or coherent reconstruction exists, run Understand Anything on the reconstruction project. It can build a graph of files, functions, classes and dependencies; it cannot make the decompilation correct. Its graph is a map of the source tree it receives, not a validation oracle.

Current DSH-specific support is not yet merged upstream as of 2026-11-23. See [Understand Anything status and workflow](UNDERSTAND-ANYTHING.md). Until that changes, complete this stage through a supported agent platform or mark it pending; do not silently install an open PR as a production plugin.

## 9. Report

Every report must contain:

1. Case ID and authorization scope.
2. Input SHA-256 and provenance.
3. Host OS, CPU architecture and tool versions.
4. Commands/workflows actually run.
5. Static findings with addresses/metadata references.
6. Dynamic findings with trace/capture identifiers.
7. Network findings with capture references and TLS/decryption status.
8. Reconstruction tests and results.
9. Understand Anything graph path/version if run.
10. Unresolved questions, uncertainty, false-positive risks and unsupported areas.

## 10. Completion criteria

A reverse-engineering task is complete only when the requested questions are answered with reproducible evidence, unverified hypotheses are labelled, artifacts are preserved, and limitations are explicit. “The model produced a plausible explanation” is not a completion criterion.
