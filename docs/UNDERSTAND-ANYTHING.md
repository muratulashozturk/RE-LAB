# Understand Anything — post-reverse-engineering stage

**Project:** [Egonex-AI/Understand-Anything](https://github.com/Egonex-AI/Understand-Anything)  
**Repository license:** MIT (project-level; verify dependency licenses for the version used)  
**Checked:** 2026-11-23

## 1. What it does

Understand Anything analyzes a codebase/documentation set and generates an interactive knowledge graph covering files, functions, classes and dependencies. It includes guided tours, semantic search, domain views and a dashboard. The default graph path is `.ua/knowledge-graph.json`; an existing `.understand-anything/` directory is retained as the data directory for that project.

Official project README: https://github.com/Egonex-AI/Understand-Anything

## 2. Where it belongs in this lab

Understand Anything is a **post-RE comprehension tool**. It is not the executable analysis stage.

Correct sequence:

1. Analyze the original target using REA and appropriate specialist engines.
2. Preserve static/dynamic/network evidence.
3. Produce a reconstructed source tree and documentation from the verified findings.
4. Run Understand Anything against that reconstructed source/documentation tree.
5. Use the graph to navigate architecture, locate dependency paths and identify missing documentation.
6. Validate any graph-derived claim against the reconstruction, original evidence and tests.

It does not decompile PE/ELF/Mach-O, reverse DEX, unpack an installer, recover symbols, attach to a process or decode a PCAP. It cannot make an inaccurate reconstruction accurate simply by graphing it.

## 3. DSH integration status — important blocker

As checked on **2026-11-23**:

- Upstream DSH support issue [#670](https://github.com/Egonex-AI/Understand-Anything/issues/670) is open.
- Proposed DSH bundle PR [#673](https://github.com/Egonex-AI/Understand-Anything/pull/673) is open and not merged into the main repository.
- The upstream platform compatibility table lists Claude Code, Cursor, VS Code/Copilot, Codex, OpenCode, Gemini CLI and other platforms, but does not list DSH as a released supported platform.
- PR #673 describes a proposed DSH bundle package and reports tests, but the open review contains a concrete concern about local-directory installation: a clean checkout lacks generated directories that the documented local install path expects unless the build/pack steps are performed. Other packaging paths also need to be reviewed.

**Decision:** Include Understand Anything in the lab's post-RE design, but do not claim it can currently be installed as a stable native DSH plugin. Do not use the unmerged PR as a production installation source. Recheck issue #670 and PR #673 before each deployment; the status may change.

This is a real integration gap, not a configuration value that can safely be guessed.

## 4. Supported use paths today

The upstream repository currently documents installers for several clients (including Codex, OpenCode, Gemini CLI, VS Code/Copilot, Cline and others) on Linux and Windows. It does not document a released DSH platform value in its compatibility table.

If you use an independently supported client as a temporary post-RE host, follow the exact current upstream installer for that client:

- Linux: `https://github.com/Egonex-AI/Understand-Anything/blob/main/install.sh`
- Windows: `https://github.com/Egonex-AI/Understand-Anything/blob/main/install.ps1`

Do not run an installer for a platform that is not listed as supported. This lab's primary agent remains DSH; using another client is an optional separate execution path, not a hidden DSH integration.

## 5. Intended project placement

Run it only on the reconstruction tree, not on the intake/originals directory. Example:

```text
cases/CASE-ID/
├── intake/originals/             # original target; do not scan as a source repo
├── evidence/                     # static/dynamic/network outputs
├── decompilation/                # tool-generated output
└── reconstruction/               # curated recovered/reimplemented source
    ├── src/
    ├── tests/
    ├── docs/
    └── .ua/                      # generated Understand Anything graph
```

Before running, remove secrets and personal data from the reconstruction tree or exclude them using the product's supported configuration. Decompiler output can contain embedded credentials or vendor data too.

## 6. Commands after DSH support is released and verified

The upstream project documents these skill commands after installation:

```text
/understand
/understand-dashboard
/understand-chat
/understand-diff
/understand-explain <path>
/understand-onboard
/understand-domain
```

Some clients use a different invocation prefix; for example, the upstream README says Codex uses `$understand`. Do not assume slash-command syntax is universal.

Expected graph file:

```text
reconstruction/.ua/knowledge-graph.json
```

Initial whole-project analysis can consume significant model tokens. Later runs are incremental by default and re-analyze changed files. This is model usage, not a license fee for the MIT project. Hosted DeepSeek usage may still be billable under the provider's current terms.

## 7. Acceptance checks before enabling it in DSH

Do not mark this integration complete until all of the following pass on the exact DSH version/profile:

- upstream DSH PR merged or a separately reviewed fork is used;
- release/package is built and its integrity/checksum recorded;
- bundle is installed into a disposable DSH profile using the documented package artifact;
- DSH reports the plugin/bundle active;
- the complete `/understand` skill set is discoverable;
- tree-sitter WASM grammars are available at runtime;
- an initial scan creates `.ua/knowledge-graph.json`;
- the dashboard opens and can read the graph;
- a second run updates only changed files as documented;
- the operation can be repeated after restarting DSH;
- all plugin code and transitive dependency licenses are checked.

Until those checks pass, label the stage `BLOCKED: DSH bundle not yet upstream-supported` in the case report.

## 8. Evidence relationship

Use the graph for navigation, not as proof. For an important node, the reconstruction documentation should link to:

- original binary address/function or source file, where available;
- Ghidra/Rizin/JADX/ILSpy artifact and tool version;
- Frida/debugger trace or packet capture if behavior is dynamic;
- test case that compares reconstructed behavior with the target.

A graph node with no evidence reference is a navigation hint only.
