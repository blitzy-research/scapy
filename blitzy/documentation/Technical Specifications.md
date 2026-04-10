# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

### 0.1.1 Core Documentation Objective

Based on the provided requirements, the Blitzy platform understands that the documentation objective is to **create new documentation** that provides a comprehensive, empirically-grounded technical explanation of Scapy's packet field calculation order, raw packet caching mechanism, cache invalidation rules, and the "dual state" behavior observed when modifying nested payloads within dissected packets.

- **Category**: Create new documentation
- **Documentation Type**: Technical deep-dive / investigative analysis document
- **Target Audience**: Developers working with custom Scapy protocol layers who encounter confusing rebuild behavior

The user's requirements translate to the following specific documentation needs:

- **Requirement 1 — Field Calculation Order**: Explain the precise ordering of field serialization during `build()`, the role of `post_build()` in computing derived values (checksums, lengths), and why the order of operations within `post_build()` matters when a checksum depends on a length field that depends on payload size.
- **Requirement 2 — show2() Consecutive Call Behavior**: Explain exactly what `show2()` does internally (`self.__class__(raw(self)).show()`), whether consecutive calls produce different results, and under what conditions the user might observe varying output.
- **Requirement 3 — The Dual-State Problem**: Explain and empirically demonstrate that modifying a payload of a sub-packet inside a `PacketListField` results in `show()` displaying the modified value while `bytes()` returns the original cached bytes — the packet existing in two simultaneous states.
- **Requirement 4 — Cache Invalidation Mechanism**: Document the complete lifecycle of `raw_packet_cache`: what sets it during dissection, what checks it during build, what invalidates it when fields change, and precisely why nested payload modifications escape detection.
- **Requirement 5 — Same-Layer vs Nested Modification**: Explain why modifying a field in the same layer where bytes were parsed triggers correct rebuild, while modifying a field in a nested payload does not.
- **Requirement 6 — The Role of copy() and clear_cache()**: Evaluate whether `copy()` resolves the dual-state problem (it does not — it preserves the cache), and document `clear_cache()` as the correct mechanism plus its caveats around dissected auto-computed field values.

### 0.1.2 Special Instructions and Constraints

- **No source code modifications**: The user explicitly states "Don't modify the repository source files." Only temporary test scripts are permitted, and they must be removed after use.
- **Implementation rule — SWE-AtlasQnA-Repo**: Create a new markdown document named `scapy_0925ada48540.md` in the `blitzy/documentation` directory that comprehensively answers all questions posed in the prompt, with rationale grounded in the source code.
- **Evidence-based**: All answers must be based on the code as the truth, with no assumptions.
- **Style**: Provide thinking and rationale behind each answer; include concrete empirical evidence from code analysis.

### 0.1.3 Technical Interpretation

These documentation requirements translate to the following technical documentation strategy:

- To document the field calculation order, we will trace the call chain from `Packet.build()` (line 746) through `do_build()` (line 724), `self_build()` (line 678), `do_build_payload()` (line 715), and `post_build()` (line 758) in `scapy/packet.py`, and show how protocol layers like `IP` (line 539 of `scapy/layers/inet.py`) and `UDP` (line 841) implement `post_build()` for auto-computed fields.
- To document the show2() behavior, we will analyze line 1486 of `scapy/packet.py` which shows `show2()` creates a temporary re-dissected packet via `self.__class__(raw(self)).show()`.
- To document the dual-state problem, we will trace `_raw_packet_cache_field_value()` (line 648) which for `PacketListField` entries only tracks `[x.fields for x in val]` — the direct `.fields` dict of each sub-packet, not their payloads.
- To document cache invalidation, we will trace `setfieldval()` (line 472), `do_dissect()` (line 1002), `self_build()` cache check (line 685), and `clear_cache()` (line 664).

### 0.1.4 Inferred Documentation Needs

Based on code analysis, the following implicit documentation needs have been identified:

- **The `do_build()` cache-vs-post_build gate**: When `raw_packet_cache` is not `None`, `do_build()` returns `pkt + pay` directly (line 740), skipping `post_build()` entirely. This means even if a payload changes and is correctly rebuilt, the parent layer's checksum (computed in `post_build`) is never recalculated when the parent's cache is still valid. This critical side effect is not documented anywhere and directly causes the user's observed behavior.
- **Dissected fields retain explicit values**: After dissection, auto-computed fields like checksums hold their parsed values (not `None`). Even after `clear_cache()`, `post_build()` guards like `if self.chksum is None` will not trigger, so checksums are not recomputed. The user must `del pkt.chksum` to reset to the default `None` value. This is an implicit requirement for correct rebuilds of modified dissected packets.
- **The `copy()` method preserves cache**: `copy()` at line 416 explicitly copies `raw_packet_cache`, so `copy()` alone does not resolve the dual-state problem. This contradicts a common user assumption.


## 0.2 Documentation Discovery and Analysis

### 0.2.1 Existing Documentation Infrastructure Assessment

Repository analysis reveals a mature Sphinx-based documentation ecosystem with protocol tutorials in Jupyter notebooks, but no existing documentation that addresses the raw packet cache lifecycle, nested modification pitfalls, or the `show()` vs `bytes()` dual-state behavior.

- **Documentation framework**: Sphinx, configured via `doc/scapy/conf.py`
- **Documentation source location**: `doc/scapy/` (RST files) and `doc/notebooks/` (Jupyter notebooks)
- **ReadTheDocs configuration**: `.readthedocs.yml` — builds on Ubuntu 20.04 with Python 3.9, publishes EPUB and PDF
- **Relevant existing documentation files examined**:
  - `doc/scapy/build_dissect.rst` — Covers the `build()` and `post_build()` pipeline, field serialization, and how to define custom protocol layers. Contains the section "Handling default values: `post_build`" which explains the general mechanism but does NOT discuss caching, cache invalidation, or the dual-state problem.
  - `doc/scapy/usage.rst` — Main practical tutorial covering interactive Scapy usage, but no coverage of caching internals.
  - `doc/scapy/functions.rst` — A worked checksum example, but limited to the computation itself, not the lifecycle of when checksums are computed vs cached.
  - `doc/scapy/advanced_usage.rst` — Covers ASN.1, automata, and PipeTools; no caching documentation.
- **API documentation tools**: None detected (no Sphinx autodoc configuration for the core `packet.py` module).
- **Diagram tools**: Mermaid (used in the technical spec), no detected PlantUML or other diagram tools in the repo.

### 0.2.2 Repository Code Analysis for Documentation

The following source files were analyzed as primary sources for answering the user's questions:

| Source File | Lines | Relevance |
|---|---|---|
| `scapy/packet.py` | 2553 | Core `Packet` class: `build()`, `do_build()`, `self_build()`, `post_build()`, `do_dissect()`, `show()`, `show2()`, `copy()`, `clear_cache()`, `setfieldval()`, `_raw_packet_cache_field_value()` |
| `scapy/fields.py` | 3868 | Field types: `PacketListField`, `FieldLenField`, `LenField`, field `addfield()`/`getfield()` methods |
| `scapy/layers/inet.py` | — | Reference implementations: `IP.post_build()` (line 539), `TCP.post_build()` (line 767), `UDP.post_build()` (line 841) showing real checksum/length computation |
| `scapy/compat.py` | — | `raw()` function (line 112) — confirms `raw(x) = bytes(x)` |
| `scapy/base_classes.py` | 510 | Metaclass machinery and `Packet_metaclass` |
| `doc/scapy/build_dissect.rst` | — | Existing documentation on the build/dissect pipeline for cross-reference |

Key code paths examined:

- **Build chain**: `build()` (L746) → `do_build()` (L724) → `self_build()` (L678) + `do_build_payload()` (L715) → `post_build()` (L758)
- **Dissection chain**: `__init__()` (L141) → `dissect()` (L1049) → `do_dissect()` (L1002) → `do_dissect_payload()` (L1023)
- **Cache population**: `do_dissect()` sets `raw_packet_cache` at line 1019 and `raw_packet_cache_fields` at lines 1005-1017
- **Cache invalidation**: `setfieldval()` clears cache at lines 484-486; `clear_cache()` recursively clears at lines 664-676
- **Cache check during build**: `self_build()` compares `raw_packet_cache_fields` at lines 685-695; `do_build()` gates `post_build()` on `raw_packet_cache is None` at lines 737-740

### 0.2.3 Web Search Research Conducted

No web search was required for this analysis. All answers are derived directly from the Scapy source code in the repository, empirical test execution, and the existing technical specification sections, as required by the "base your answers on the code as the truth" instruction.


## 0.3 Documentation Scope Analysis

### 0.3.1 Code-to-Documentation Mapping

The user's questions target five interconnected mechanisms all residing in `scapy/packet.py`, with supporting behavior in `scapy/fields.py` and protocol examples in `scapy/layers/inet.py`. Each mechanism maps to a specific documentation section in the output document.

- **Module: `scapy/packet.py` — `Packet` class**
  - Public APIs requiring documentation coverage:
    - `build()` (line 746), `do_build()` (line 724), `self_build()` (line 678)
    - `post_build()` (line 758), `do_build_payload()` (line 715)
    - `do_dissect()` (line 1002), `dissect()` (line 1049)
    - `show()` (line 1459), `show2()` (line 1473)
    - `copy()` (line 407), `clear_cache()` (line 664)
    - `setfieldval()` (line 472), `delfieldval()` (line 504)
    - `_raw_packet_cache_field_value()` (line 648)
    - `__div__()` / `__truediv__()` (line 596) — layer stacking
    - `__iter__()` / `clone_with()` — volatile value resolution
  - Current documentation: `doc/scapy/build_dissect.rst` covers `build()`, `do_build()`, `post_build()` at a surface level but does NOT document caching, `raw_packet_cache`, or the `do_build()` cache gate.
  - Documentation needed: Comprehensive explanation of the cache lifecycle, cache invalidation rules, nested modification behavior, and practical remedies.

- **Module: `scapy/fields.py` — `PacketListField` class**
  - Relevant APIs: `PacketListField.addfield()` (line 1785), `PacketListField.getfield()` (line 1721)
  - Current documentation: Missing from any existing doc regarding cache tracking behavior.
  - Documentation needed: Explanation of how `_raw_packet_cache_field_value()` only tracks `.fields` dicts of sub-packets (not their payloads), creating the cache detection blind spot.

- **Module: `scapy/layers/inet.py` — `IP`, `TCP`, `UDP` protocol layers**
  - Relevant APIs: `IP.post_build()` (line 539), `TCP.post_build()` (line 767), `UDP.post_build()` (line 841)
  - Documentation needed: Used as canonical reference examples showing proper `post_build()` implementation patterns (length before checksum).

### 0.3.2 Documentation Gap Analysis

Given the requirements and repository analysis, the following documentation gaps are identified:

- **Undocumented raw_packet_cache lifecycle**: No existing documentation explains when `raw_packet_cache` is set (during dissection), what it stores (the original wire bytes for the current layer), how it is checked during `self_build()`, or how it gates `post_build()` in `do_build()`.
- **Undocumented cache invalidation rules**: No documentation explains that `setfieldval()` clears the cache only for the layer where the field is defined, and that payload field modifications delegate to the payload's `setfieldval()` without clearing the parent's cache.
- **Undocumented PacketListField cache blind spot**: The behavior where `_raw_packet_cache_field_value()` tracks only `[x.fields for x in val]` for packet-holding fields, ignoring payload modifications, is not documented anywhere.
- **Undocumented show() vs show2() vs bytes() behavioral differences**: No documentation explains that `show()` reads in-memory field values directly, `show2()` builds-then-re-dissects, and `bytes()` may return cached bytes from dissection.
- **Undocumented copy() cache preservation**: The fact that `copy()` preserves `raw_packet_cache` (line 416) is not documented as a potential source of confusion.
- **Undocumented clear_cache() + del field pattern**: The correct pattern for fully rebuilding a modified dissected packet (clear cache then delete auto-computed fields) is not documented.


## 0.4 Documentation Implementation Design

### 0.4.1 Documentation Structure Planning

The output document `blitzy/documentation/scapy_0925ada48540.md` will be structured as a single comprehensive Markdown document organized into the following hierarchy:

```
blitzy/documentation/scapy_0925ada48540.md
├── Title and Preamble
├── 1. Field Calculation Order in post_build()
│   ├── The Build Pipeline
│   ├── How post_build() Receives Fully-Built Payload
│   ├── Why Order Within post_build() Matters
│   └── Canonical Examples (IP, TCP, UDP)
├── 2. show2() Behavior on Consecutive Calls
│   ├── What show2() Actually Does
│   ├── Why Consecutive Calls Produce Identical Results
│   └── show() vs show2() vs bytes() Comparison
├── 3. The Dual-State Problem
│   ├── Reproducing the Behavior
│   ├── Root Cause: Cache Tracking Blind Spot
│   ├── Same-Layer vs Nested Modification
│   └── Why show() and bytes() Disagree
├── 4. The Complete Cache Lifecycle
│   ├── Cache Population During Dissection
│   ├── Cache Check During Build
│   ├── Cache Invalidation Rules
│   └── The do_build() Cache Gate
├── 5. Does copy() Fix the Dual-State Problem?
│   ├── What copy() Does to the Cache
│   ├── Why copy() Does Not Help
│   └── The Correct Fix: clear_cache() + del
└── 6. Summary of Practical Remedies
```

### 0.4.2 Content Generation Strategy

- **Information Extraction Approach**:
  - Extract the exact method implementations from `scapy/packet.py` and `scapy/fields.py` to trace the complete build, dissect, and cache-check call chains.
  - Generate empirical examples by constructing custom protocol classes that isolate each behavior in question.
  - Create Mermaid diagrams by mapping the call flow from `build()` through `post_build()`, and the cache lifecycle from `do_dissect()` through `self_build()`.

- **Documentation Standards**:
  - Markdown formatting with hierarchical headers (`#`, `##`, `###`)
  - Mermaid diagram integration using fenced code blocks
  - Code examples using Python fenced blocks with inline comments
  - Source citations as inline references: `Source: scapy/packet.py:LineNumber`
  - All claims grounded in specific source code lines

### 0.4.3 Diagram and Visual Strategy

The following Mermaid diagrams will be included:

- **Build pipeline flowchart**: Showing the call chain from `build()` → `do_build()` → `self_build()` → `do_build_payload()` → `post_build()` with the cache gate decision point.
- **Dissection cache population flowchart**: Showing how `do_dissect()` populates `raw_packet_cache` and `raw_packet_cache_fields`.
- **Cache invalidation comparison diagram**: Side-by-side flow showing same-layer modification (cache cleared) vs nested modification (cache preserved).
- **Dual-state sequence diagram**: Showing how `show()` reads in-memory values while `bytes()` returns cached bytes for the same packet object.


## 0.5 Documentation File Transformation Mapping

### 0.5.1 File-by-File Documentation Plan

| Target Documentation File | Transformation | Source Code/Docs | Content/Changes |
|---|---|---|---|
| `blitzy/documentation/scapy_0925ada48540.md` | CREATE | `scapy/packet.py`, `scapy/fields.py`, `scapy/layers/inet.py`, `scapy/compat.py`, `doc/scapy/build_dissect.rst` | Comprehensive investigative document answering all user questions about field calculation order, `show2()` behavior, the dual-state problem with `PacketListField` nested payloads, cache invalidation mechanics, and the role of `copy()` vs `clear_cache()`. Includes Mermaid diagrams, code-traced rationale, and empirical evidence. |

No existing files are modified or deleted per the "SWE-AtlasQnA-Repo" rule and the user's explicit instruction not to modify repository source files.

### 0.5.2 New Documentation File Detail

```
File: blitzy/documentation/scapy_0925ada48540.md
Type: Technical Deep-Dive / Investigative Q&A
Source Code:
  - scapy/packet.py (primary — lines 141-1486)
  - scapy/fields.py (lines 648-662, 1562-1787)
  - scapy/layers/inet.py (lines 521-864)
  - scapy/compat.py (line 112)
Sections:
  - Field Calculation Order: traces build() → do_build() → self_build() → post_build()
  - show2() Analysis: explains self.__class__(raw(self)).show() at line 1486
  - Dual-State Problem: traces _raw_packet_cache_field_value() at line 648
    and the cache gate at do_build() lines 737-740
  - Cache Lifecycle: covers do_dissect() cache setup (lines 1002-1021),
    self_build() cache check (lines 685-695), setfieldval() invalidation (lines 484-486)
  - copy() vs clear_cache(): traces copy() at line 407 (preserves cache)
    and clear_cache() at line 664 (recursively clears)
  - Practical Remedies: clear_cache() + del for auto-computed fields
Diagrams:
  - Build pipeline flowchart with cache gate
  - Dissection cache population flow
  - Cache invalidation comparison (same-layer vs nested)
Key Citations:
  - scapy/packet.py lines 141, 407, 472, 484-486, 596, 648-662, 664-676,
    678-695, 715-740, 758, 1002-1021, 1473-1486
  - scapy/fields.py lines 648-662, 1562-1787
  - scapy/layers/inet.py lines 539-551, 767-786, 841-864
```

### 0.5.3 Documentation Configuration Updates

No documentation configuration changes are required. The output document is a standalone Markdown file placed in `blitzy/documentation/` per the SWE-AtlasQnA-Repo implementation rule. It does not integrate into the Sphinx documentation build (`doc/scapy/`), mkdocs, or any other documentation framework.


## 0.6 Dependency Inventory

### 0.6.1 Documentation Dependencies

No additional documentation tools or packages are required for this task. The output is a standalone Markdown file with embedded Mermaid diagram syntax, requiring no build step.

| Registry | Package Name | Version | Purpose |
|---|---|---|---|
| pip | scapy | 2.7.0+ (local repo) | The subject of documentation; installed from local repository for empirical verification |
| (runtime) | Python | ≥ 3.7, < 4 (3.12.3 used) | Required runtime per `pyproject.toml` `requires-python` field |

The Scapy project itself declares no mandatory external dependencies (`README.md`: "Scapy does not require any external Python modules on Linux and BSD-like systems"). The project uses Sphinx for its documentation build (`doc/scapy/conf.py`), but this task does not modify or build the Sphinx documentation.

### 0.6.2 Documentation Reference Updates

Not applicable. This task creates a new standalone document and does not modify any existing documentation files or links.


## 0.7 Coverage and Quality Targets

### 0.7.1 Documentation Coverage Metrics

The user posed six distinct technical questions. The target is 100% coverage of all questions with evidence-based answers grounded in specific source code lines.

| Question | Coverage Target | Source Evidence Required |
|---|---|---|
| Field calculation order: why is checksum computed before length is finalized? | Complete explanation of `build()` pipeline and `post_build()` execution order | `scapy/packet.py` lines 724-767, `scapy/layers/inet.py` lines 539-551 |
| Why does calling `show2()` twice produce correct checksum on second call? | Complete explanation with analysis of whether this actually occurs | `scapy/packet.py` lines 1473-1486, lines 724-740 |
| Why does `bytes()` return original bytes while `show()` shows modified values? | Root-cause analysis of `_raw_packet_cache_field_value()` cache blind spot | `scapy/packet.py` lines 648-662, 685-695, 737-740 |
| What determines whether Scapy rebuilds vs returns cached bytes? | Complete cache lifecycle documentation | `scapy/packet.py` lines 1002-1021, 678-695, 484-486, 664-676 |
| How does modifying a nested payload differ from modifying a direct field? | Comparison of cache invalidation paths | `scapy/packet.py` lines 472-492, 648-662 |
| Does `copy()` fix the dual-state problem? Why or why not? | Analysis of `copy()` cache behavior and correct alternative | `scapy/packet.py` lines 407-426, 664-676 |

### 0.7.2 Documentation Quality Criteria

- **Completeness**: Every question must receive a definitive, code-traced answer with line-number citations.
- **Accuracy**: All claims must be verified against the source code. Empirical test results (run during context gathering) confirm the documented behavior.
- **Clarity**: Explanations must follow a progressive structure — observed behavior first, then root cause, then remedy.
- **Diagrams**: Mermaid flowcharts and sequence diagrams required for the build pipeline, cache lifecycle, and dual-state behavior.
- **No assumptions**: Per the implementation rule, all answers must be grounded in code, not theoretical reasoning.

### 0.7.3 Example and Diagram Requirements

- Minimum of one code example per major concept (field calculation order, dual-state reproduction, cache clearing)
- Four Mermaid diagrams: build pipeline, dissection cache population, cache invalidation comparison, and dual-state illustration
- All code examples must use custom protocol classes (not Scapy built-in protocols) to isolate the demonstrated behavior from protocol-specific complexity


## 0.8 Scope Boundaries

### 0.8.1 Exhaustively In Scope

- **New documentation file**:
  - `blitzy/documentation/scapy_0925ada48540.md` — The sole deliverable; a comprehensive investigative document answering all user questions
- **Source code analysis** (read-only, no modifications):
  - `scapy/packet.py` — Core `Packet` class: build, dissect, cache, copy, show methods
  - `scapy/fields.py` — Field types, especially `PacketListField` and cache tracking helpers
  - `scapy/layers/inet.py` — `IP`, `TCP`, `UDP` `post_build()` reference implementations
  - `scapy/compat.py` — `raw()` function definition
  - `scapy/base_classes.py` — Metaclass machinery
  - `doc/scapy/build_dissect.rst` — Existing documentation for cross-reference
- **Empirical verification** (temporary test scripts, removed after use):
  - Custom protocol classes to reproduce the dual-state behavior
  - Verification of `copy()` vs `clear_cache()` behavior
  - Verification of `show2()` consecutive call behavior
  - Verification of same-layer vs nested modification cache invalidation

### 0.8.2 Explicitly Out of Scope

- **Source code modifications**: No files in the Scapy repository will be modified (per explicit user instruction and SWE-AtlasQnA-Repo rule)
- **Test file modifications**: No test files will be created, modified, or committed
- **Feature additions or bug fixes**: The dual-state behavior is documented as-is; no patches are proposed for `_raw_packet_cache_field_value()` or `setfieldval()`
- **Sphinx documentation integration**: The output document is NOT added to `doc/scapy/index.rst` or the Sphinx build configuration
- **Other protocol layers**: Only `IP`, `TCP`, and `UDP` are referenced as canonical `post_build()` examples; no analysis of contrib protocols, TLS, or automotive layers
- **Deployment or CI changes**: No changes to `.readthedocs.yml`, `tox.ini`, or CI configuration
- **Performance analysis**: While caching is a performance optimization, this document does not benchmark or profile cache performance


## 0.9 Execution Parameters

### 0.9.1 Documentation-Specific Instructions

- **Documentation build command**: Not applicable — the output is a standalone Markdown file, not part of the Sphinx build.
- **Documentation preview command**: Any Markdown renderer (e.g., VS Code preview, GitHub rendering, `grip` CLI tool).
- **Diagram generation**: Mermaid diagrams are embedded inline in the Markdown; renderers that support Mermaid (GitHub, GitLab, Mermaid Live Editor) will display them natively.
- **Default format**: Markdown with Mermaid diagrams.
- **Citation requirement**: Every technical claim must reference a specific source file and line number.
- **Style guide**: Follows the SWE-AtlasQnA-Repo rule — provide thinking/rationale behind answers, base everything on the code as truth, and do not make assumptions.
- **Output location**: `blitzy/documentation/scapy_0925ada48540.md` (per implementation rule naming convention: `<source_branch_name>.md`).
- **Documentation validation**: Manual review of all code citations against the repository; all empirical claims were verified by executing temporary test scripts during context gathering.


## 0.10 Rules for Documentation

The following rules apply to this documentation task, derived from the user's explicit instructions and the SWE-AtlasQnA-Repo implementation rule:

- **Do not modify any existing files in the source repository.** The output is a new file in `blitzy/documentation/` only.
- **Create a new markdown document named `scapy_0925ada48540.md`** that comprehensively answers all questions posed in the prompt.
- **Provide thinking and rationale behind the answers.** Each answer must explain the "why" through code-traced reasoning, not just state the "what."
- **Do not make assumptions; base answers on the code as the truth.** Every claim must be traceable to a specific line in the source code.
- **Place the generated document in the `blitzy/documentation` directory** in the destination repo.
- **Temporary test scripts are permitted but must be removed when done.** All empirical verification scripts created during context gathering have been deleted.
- **No source code modifications.** This is a documentation-only task; no changes to `scapy/packet.py`, `scapy/fields.py`, or any other file in the repository.


## 0.11 References

### 0.11.1 Source Files Analyzed

The following files and directories were searched and analyzed to derive the conclusions in this Agent Action Plan:

| File/Directory | Purpose of Analysis |
|---|---|
| `scapy/packet.py` (lines 1-1930) | Core `Packet` class — build, dissect, cache, copy, show, setfieldval, clear_cache, `_raw_packet_cache_field_value`, `__div__`, `clone_with`, `__iter__`, `NoPayload`, `Raw`, `Padding` |
| `scapy/fields.py` (lines 1562-1787, 2086-2204) | `PacketListField` class (dissection, serialization, cache tracking), `FieldLenField` (`i2m` auto-computation), `LenField` (payload length auto-computation) |
| `scapy/layers/inet.py` (lines 521-864) | `IP.post_build()`, `TCP.post_build()`, `UDP.post_build()` — canonical reference implementations for auto-computing length, checksum, and data offset in `post_build()` |
| `scapy/compat.py` (lines 112-118) | `raw()` function confirming `raw(x) = bytes(x)` |
| `scapy/base_classes.py` (lines 1-510) | Metaclass machinery, `Packet_metaclass` |
| `doc/scapy/build_dissect.rst` (lines 520-841) | Existing Sphinx documentation on the build/dissect pipeline, `post_build()` explanation, and build order trace |
| `doc/scapy/conf.py` | Sphinx configuration confirming documentation framework |
| `doc/scapy/` (directory) | Full documentation source tree — verified no existing doc covers caching internals |
| `pyproject.toml` | Project metadata: Python requirement (≥3.7, <4), version sourcing, package registry |
| `tox.ini` | Testing matrix: Python 3.4–3.11 supported |
| `.readthedocs.yml` | ReadTheDocs configuration: Ubuntu 20.04, Python 3.9 |
| `README.md` | Project overview, zero-dependency claim, supported platforms |
| Root directory (`""`) | Repository structure assessment |
| `scapy/` (directory) | Package structure: layers, contrib, arch, asn1, libs, modules, tools |
| `doc/` (directory) | Documentation ecosystem: notebooks, Sphinx source, Vim syntax, Vagrant CI |

### 0.11.2 Technical Specification Sections Referenced

| Section | Content Used |
|---|---|
| 1.1 Executive Summary | Project overview, version (2.7.0), Python requirements, license |
| 4.2 CORE PACKET ENGINE PROCESSES | Build lifecycle flowchart, dissection lifecycle, layer stacking mechanism |
| 5.2 COMPONENT DETAILS | Core Packet Engine interfaces, `raw_packet_cache` description, build/dissect sequence diagrams |

### 0.11.3 Empirical Verification Conducted

The following temporary test scripts were executed during context gathering and subsequently removed:

| Test | Purpose | Key Finding |
|---|---|---|
| `test_field_order.py` | Verify field serialization order in `self_build()` | Fields are processed in `fields_desc` order; `post_build()` receives fully built payload |
| `test_show2_issue.py` | Verify `show2()` behavior for constructed packets | `show2()` produces consistent results across calls; `show()` displays `None` for auto fields |
| `test_cache_nested.py` | Test cache with direct field modification in `PacketListField` sub-packets | Direct field changes in sub-packets ARE detected by the cache comparison |
| `test_nested_payload2.py` | Test cache with mutable field modification in `PacketListField` sub-packets | Mutable field changes (e.g., `item_data`) are detected because `_raw_packet_cache_field_value` tracks them |
| `test_payload_chain.py` | **KEY TEST** — Modify PAYLOAD of a sub-packet in `PacketListField` | `show()` displays modified value; `bytes()` returns original bytes — DUAL STATE CONFIRMED |
| `test_copy_fix.py` | Test whether `copy()` resolves the dual-state problem | `copy()` PRESERVES the cache — does NOT fix the problem; `clear_cache()` IS the fix |
| `test_same_layer.py` | Compare same-layer vs nested modification on regular payload chains | Both work correctly for regular payload chains (not `PacketListField`); nested payload's cache is cleared, parent layer uses own cache but child rebuilds |
| `test_nested_checksum.py` | Test checksum behavior when payload is modified in a regular chain | Parent layer's cache gate skips `post_build()`, so checksum is NOT recomputed; even `clear_cache()` is insufficient because dissected checksum value is explicit (not None) |
| `test_full_lifecycle.py` | Test complete fix: `clear_cache()` + `del pkt.chksum` | Deleting the checksum field resets it to `None`, allowing `post_build()` to recompute it correctly |
| `test_show2_consecutive.py` | Verify `show2()` idempotency on constructed packets | Consecutive `show2()` calls produce identical results; no state mutation |
| `test_show2_modified.py` | Verify `show2()` on dissected packet with nested `PacketListField` modification | `show2()` returns ORIGINAL cached bytes (not modified values) until `clear_cache()` is called |

### 0.11.4 Attachments

No attachments were provided by the user for this project.


