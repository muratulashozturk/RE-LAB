# Licenses, scope and exclusions

**Policy:** include free/open-source software only. **Checked:** 2026-11-23.  
The table records upstream project license metadata as checked; it is not a legal review of every transitive dependency, binary redistribution right, SDK agreement, plugin or extension.

## 1. Selected components

| Component | Upstream project | Project license checked | Use in RE-LAB | Notes |
|---|---|---|---|---|
| REA | https://github.com/morluto/rea | MIT | AI RE orchestration | Third-party dependencies retain their own licenses. |
| LASIK | https://github.com/pktq/lasik | MIT | Static anti-BLA preprocessing | Never executes input according to project documentation. |
| ReClarity | https://github.com/i098/ReClarity | MIT | Optional anti-BLA/coverage workflow | Needs Bun; `bwrap` for coverage. |
| Understand Anything | https://github.com/Egonex-AI/Understand-Anything | MIT | Post-RE codebase graph | DSH integration not yet upstream-supported at checked date. |
| Ghidra | https://github.com/NationalSecurityAgency/ghidra | Apache-2.0 | Static decompilation/disassembly | Ghidra 12.2 requires JDK 25. Check bundled components/extensions separately. |
| ghidra-rpc | https://github.com/cellebrite-labs/ghidra-rpc | MIT | Optional CLI/RPC access to Ghidra | Not an MCP server. |
| Frida | https://github.com/frida/frida | wxWindows Library Licence 3.1 (project components vary) | Dynamic instrumentation | Audit the exact CLI/server/gadget component and dependencies being deployed. |
| x64dbg | https://github.com/x64dbg/x64dbg | GPL family; component licenses vary | Windows debugger | Check bundled plugin/components individually. |
| Rizin | https://github.com/rizinorg/rizin | LGPL-3.0 | Binary analysis CLI | Optional second opinion. |
| Rizin MCP | https://github.com/rizinorg/rizin-mcp | LGPL-3.0 and other component licenses | Optional MCP interface | Review repository's license files and linked dependencies. |
| Cutter | https://github.com/rizinorg/cutter | GPL-3.0 | Rizin GUI | Optional. |
| JADX | https://github.com/skylot/jadx | Apache-2.0 | Android/DEX decompilation | Verify release and bundled plugin dependencies. |
| Apktool | https://github.com/iBotPeaches/Apktool | Apache-2.0 | Android resources/smali | Verify release and dependencies. |
| ILSpy | https://github.com/icsharpcode/ILSpy | MIT | .NET decompilation | Verify `ilspycmd` package version. |
| CFR | https://github.com/leibnitz27/cfr | MIT | Java bytecode decompilation | Download release from the upstream repository. |
| Wireshark | https://github.com/wireshark/wireshark | GPL-2.0 | Packet analysis | Npcap and other capture dependencies have their own terms. |
| mitmproxy | https://github.com/mitmproxy/mitmproxy | MIT | HTTP(S)/WebSocket proxy analysis | Current release requires a suitable modern Python; use isolated Python 3.12. |
| Scapy | https://github.com/secdev/scapy | GPL-2.0 | Packet analysis/experiments | Optional; use only where protocol experimentation is authorized. |
| angr | https://github.com/angr/angr | BSD-2-Clause | Symbolic execution | Optional advanced analysis; Python dependencies need their own review. |
| Playwright | https://github.com/microsoft/playwright | Apache-2.0 | Browser automation and testing | Browser binaries and downloaded runtime components should be checked separately. |
| YARA | https://github.com/VirusTotal/yara | BSD-3-Clause | Pattern matching | Optional. Rule sets have separate licenses. |
| capa | https://github.com/mandiant/capa | Apache-2.0 | Capability identification | Rules and external signatures may have distinct terms. |

License identifiers can change in newer versions or differ between repository and packaged artifacts. Recheck the exact release/tag before deployment and retain its `LICENSE`, `NOTICE`, dependency license report and source attribution where required.

## 2. Components not in the approved baseline

- **IDA Pro, Hopper, Binary Ninja, JEB:** excluded because they are commercial/proprietary products, not FOSS baseline tools.
- **Reqable:** not included because this document has not verified a suitable upstream FOSS license for the exact product/distribution. Use mitmproxy and Wireshark instead.
- **Unverified MCP servers/plugins:** not added merely because they are on GitHub. Check repository license, packaged code, dependencies, tool permissions and DSH compatibility.

## 3. Android SDK and platform tools

ADB/platform-tools are distributed by Google under their applicable SDK/platform-tools terms. Verify the license terms for the exact binary package and distribution use. Do not assume that all Android SDK components are interchangeable with an OSI-licensed source dependency.

## 4. DeepSeek model and inference

The RE-LAB code/tool selection being FOSS does not make the DeepSeek model weights, hosted API, data handling, quota or inference free/open-source. Record:

- model identifier and provider;
- hosted API versus local inference;
- provider pricing/quotas and current usage terms;
- whether submitted target code/evidence leaves the local environment;
- retention and data-processing terms that apply to the account.

Do not send confidential source, keys, production captures or personal data to a hosted model unless the owner has approved that disclosure.

## 5. FOSS acceptance gate

Before approving a new component:

1. Identify the exact upstream repository and immutable release/tag/commit.
2. Read the license file and any `NOTICE`.
3. Generate/check a dependency license report for packaged dependencies.
4. Review binary downloads and bundled components separately from source license.
5. Record whether the tool sends code, samples or telemetry to a remote service.
6. Record the approved version and checksum in this file or a case manifest.

A public GitHub repository is not automatically open source. An MCP server is not automatically safe because it uses stdio. A permissive top-level license does not erase dependency obligations.
