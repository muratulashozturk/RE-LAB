# Validation and benchmark

This file defines what must be tested before calling RE-LAB installed or more effective than REA + ReClarity. No benchmark results are claimed here; fill in the result section with measurements from your machines.

## 1. Installation readiness

### Windows 10

```powershell
node --version
npm --version
python --version
git --version
java --version
rea --version
rea doctor --client dsh
frida --version
frida-ps --local
adb version
ilspycmd --help
```

If DSH does not run on Windows, skip `rea doctor --client dsh` on Windows and validate on the DSH host instead.

### Ubuntu 22.04

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

The expected result is not just that commands exist. Save the full doctor output and check provider `ready`/diagnostic status.

## 2. Ghidra version and current known issue

As checked on 2026-11-23:

- Ghidra 12.2 was released in October 2026.
- Ghidra 12.2 requires 64-bit JDK 25.
- REA's current provider documentation accepts Ghidra 12.2 or later, and its Linux support is documented for x64/arm64. Windows x64 is documented as experimental P0.
- REA's provider notes a specific Ghidra 12.2.0 Clang `-O0` x86-64 switch/jump-table flow-reference defect, tracked as Ghidra issue #9996 and described as clearing on a later 12.2 patch. The provider notes also document an AArch64 byte-offset-switch recovery issue (#10154) as unresolved at the checked revision.

References:

- [Ghidra 12.2 change history](https://www.ghidradocs.com/12.2_PUBLIC/docs/ChangeHistory.html)
- [Ghidra 12.2 getting started](https://github.com/NationalSecurityAgency/ghidra/blob/Ghidra_12.2_build/GhidraDocs/GettingStarted.md)
- [REA Ghidra provider documentation](https://github.com/morluto/rea/blob/main/docs/getting-started/installation.md#ghidra)
- [Ghidra issue #9996](https://github.com/NationalSecurityAgency/ghidra/issues/9996)
- [Ghidra issue #10154](https://github.com/NationalSecurityAgency/ghidra/issues/10154)

For affected compiler/architecture combinations, check jump tables and control flow manually in assembly and cross-check with Rizin and runtime evidence. Do not rely only on decompiler pseudocode.

## 3. Functional smoke tests

Use binaries and applications that you own or are deliberately created for the lab. Keep tests small and deterministic.

| ID | Smoke test | Pass condition |
|---|---|---|
| W-01 | Analyze a harmless PE test program in Ghidra GUI | Correct architecture and imports shown; function can be disassembled/decompiled |
| W-02 | Attach x64dbg to a self-built Windows test program | Breakpoint hit and process exits normally |
| W-03 | Run `frida-ps --local` and instrument a self-built test process | Local process enumeration works; hook result recorded |
| L-01 | Analyze an ELF test program in Ghidra/Rizin | ELF architecture, symbols/imports and one function recovered |
| L-02 | Run GDB on a self-built Linux test program | Breakpoint and stack inspection work |
| L-03 | Run REA Ghidra provider doctor on Ubuntu | Provider explicitly reports ready |
| N-01 | Capture a local HTTP test request with Wireshark | Correct packets/interface/time window captured |
| N-02 | Proxy a test HTTP(S) client through mitmproxy | Request/response visible where client trust and proxy configuration allow |
| A-01 | Decompile a test APK with JADX and Apktool | Package, manifest, resources and expected test class recovered |
| D-01 | Decompile a test .NET assembly with ILSpy | Test class/method visible and output is readable |
| J-01 | Decompile a test JAR with CFR | Test class and method recovered |
| E-01 | Open a test web/Electron project in Playwright/DevTools | Deterministic test action can be reproduced |
| B-01 | Scan a controlled BLA fixture with LASIK | Detection report generated; original hash unchanged |
| B-02 | Run ReClarity coverage on a supported Linux fixture | `bwrap`-isolated run completes and coverage evidence is stored |
| UA-01 | Run Understand Anything on reconstructed source via a supported host | `.ua/knowledge-graph.json` created and dashboard reads it |
| DSH-01 | Restart DSH after REA setup | `mcp__rea__...` tools are discoverable and one test call succeeds |

## 4. Comparative benchmark against REA + ReClarity

Do not compare a larger stack using a different model, different prompt, different analysis budget or different target and then call the result fair.

Use the same target files and same question set. Run each configuration at least three times if the model is nondeterministic and report median plus range. If resources do not allow three runs, label it a smoke test, not a benchmark.

Recommended configurations:

| Configuration | Description |
|---|---|
| A | REA only |
| B | REA + ReClarity |
| C | REA + LASIK (only where BLA is detected or seeded in a test fixture) |
| D | REA + Ghidra + Frida (where native/dynamic behavior applies) |
| E | Full applicable RE-LAB stack |
| F | E + Understand Anything on the reconstruction tree (post-RE comprehension metric only) |

Suggested test matrix:

| Target class | Test question category | Ground truth |
|---|---|---|
| Windows PE | function behavior, imports, control flow | source-built test program or developer-known symbols |
| Linux ELF | function behavior, call graph, syscalls | source-built test program |
| Electron | IPC channels, module boundaries, API calls | source application and captured test flows |
| HTTP/WS API | method, path, body schema, state transitions | controlled test server logs/spec |
| Android APK | component entry points, permissions, API flow | source-built test APK |
| iOS IPA | app metadata and statically recovered behavior | source-built app; runtime coverage only where device workflow is available |
| .NET/Java | method/class reconstruction | source-built assemblies/JAR |

Metrics:

- exact answer accuracy against known ground truth;
- function/endpoint/class mapping precision and recall;
- false positive and false negative count;
- behavioral equivalence test pass rate;
- analysis wall time;
- token/inference cost;
- number of tool calls and failed calls;
- reproducibility between runs;
- analyst time needed to validate output.

Understand Anything should be evaluated separately for navigation/comprehension time and graph coverage. It should not be scored as an executable decompiler.

## 5. Result template

```yaml
benchmark_id: BENCH-ID
run_date: "YYYY-MM-DD"
model: "exact model identifier"
dsh_version: "exact version or unknown"
rea_version: "exact version"
operating_host: "Windows 10 or Ubuntu 22.04"
configuration: "A/B/C/D/E/F"
target_sha256: "..."
question_set_version: "..."
repetitions: 3
results:
  accuracy: null
  false_positives: null
  false_negatives: null
  behavior_tests_passed: null
  duration_seconds: null
  inference_cost: null
  failed_tool_calls: null
notes: []
```

Do not fill unknown metrics with estimates.

## 6. Completion rule

Only call a component installed after its verification test passes on the intended host. Only call the stack better than REA + ReClarity after a controlled comparison demonstrates an improvement on the actual target classes and cost/time constraints.
