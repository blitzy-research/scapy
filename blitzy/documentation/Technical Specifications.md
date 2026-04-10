# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

Based on the provided requirements, the Blitzy platform understands that the documentation objective is to **create new documentation** that comprehensively answers a series of exploratory and analytical questions about the Scapy packet manipulation library, built from a specific commit (`0925ada48540`). The user seeks a single, consolidated Markdown document placed in `blitzy/documentation/scapy_0925ada48540.md` that serves as both an educational guide and a verified reference, grounded entirely in observed behavior and source code analysis.

**Category:** Create new documentation
**Documentation type:** Technical Q&A / Exploratory Reference Guide

### 0.1.1 Core Documentation Objectives

The user's requirements decompose into six distinct documentation goals:

- **Scapy Shell Startup Documentation** — Launch the Scapy interactive console and document the startup banner, ASCII art logo, version display, welcome message, and random quote. Record the version string returned by `conf.version` and explain how the version is resolved through the four-method fallback chain in `scapy/__init__.py` (environment variable → VERSION file → git archive → git describe → file timestamp → `0.0.0`).
- **ICMP Echo Request Construction** — Create a basic `IP()/ICMP()` packet and document every field in the IP header, distinguishing between fields with explicit defaults (e.g., `version=4`, `ttl=64`, `id=1`) and fields that are `None` before build and auto-computed during `post_build()` (e.g., `ihl`, `len`, `chksum`). Show the exact output of `pkt.show()`.
- **IP Header Auto-Population Explanation** — Walk through the source code to explain how Scapy's `IP.post_build()` in `scapy/layers/inet.py` (line 539) automatically calculates `ihl`, `len`, and `chksum` when their values are `None`. Explain the role of `SourceIPField` (in `scapy/fields.py`, line 854) and `DestIPField` (in `scapy/layers/inet.py`, line 503) in resolving source and destination addresses via the routing table.
- **ICMP Packet Transmission to Localhost** — Send a crafted `ICMP` echo-request to `127.0.0.1` using `sr1()` and document the network-layer behavior: the socket type used, the stimulus-response matching, and the echo-reply received. Describe what happens at each layer (IP construction → socket transmission → kernel processing → ICMP echo-reply → dissection).
- **Source Code Walkthrough for IP Header Construction** — Trace the packet build pipeline from `Packet.build()` (line 746) → `do_build()` (line 724) → `self_build()` (line 678) → `post_build()` (line 758) in `scapy/packet.py`, and then into `IP.post_build()` in `scapy/layers/inet.py` (line 539). Include the `bind_layers(IP, ICMP, frag=0, proto=1)` binding that auto-sets the `proto` field.
- **Test Suite Execution and Summary** — Run the Scapy UTscapy test suite using the Linux non-root configuration (`test/configs/linux.utsc`) and document the total number of test cases, campaigns, and the pass/fail breakdown.

### 0.1.2 Special Instructions and Constraints

- **No source file modifications** — The user explicitly states: "Don't modify any source files." All exploration must be read-only. The implementation rule reinforces: "Do not modify any existing files in the source repository."
- **Temporary script cleanup** — If temporary scripts are needed, they must be cleaned up after use.
- **Output placement** — Per the implementation rule `SWE-AtlasQnA-Repo`, the generated document must be named `scapy_0925ada48540.md` (matching the branch name) and placed in the `blitzy/documentation/` directory.
- **Evidence-based answers** — "Do not make assumptions, base your answers on the code as the truth." Every claim must be traceable to specific source files and line numbers.
- **Rationale required** — "Provide thinking / rationale behind the answers." Each answer must include explanatory reasoning, not just raw output.

### 0.1.3 Technical Interpretation

These documentation requirements translate to the following technical documentation strategy:

- To document the Scapy shell startup, we will **create** a section in `blitzy/documentation/scapy_0925ada48540.md` that captures the full banner output from `scapy/main.py` (lines 593–651), including the ASCII logo, version string, GitHub URL, and a random quote from the `QUOTES` list (line 47).
- To document the ICMP echo request, we will **create** a section showing the exact output of `IP()/ICMP()` construction including `repr()`, `show()`, and field-by-field analysis, referencing `IP.fields_desc` at line 524 of `scapy/layers/inet.py` and `ICMP.fields_desc` at line 954.
- To explain IP header auto-population, we will **create** a source-code walkthrough section tracing through `Packet.build()` → `self_build()` → `IP.post_build()`, documenting how `ihl`, `len`, and `chksum` are computed, and how `SourceIPField.__findaddr()` resolves the source IP via `conf.route.route()`.
- To document ICMP transmission, we will **create** a section describing the `sr1()` flow from `scapy/sendrecv.py` (line 656) through `L3RawSocket` to the kernel and back, with the actual echo-reply output.
- To document the test suite, we will **create** a section reporting the full UTscapy run against `test/configs/linux.utsc` with the `-N` (non-root) and `-b` (don't stop at first failure) flags, summarizing 190 test campaigns across 193 `.uts` files containing 5,281 individual test cases.

### 0.1.4 Inferred Documentation Needs

Based on repository analysis, the following implicit documentation needs have been identified:

- **Version resolution explanation** — The user asks "What version is it running?" but the commit `0925ada48540` has no git tags, so the version falls back to the file modification date format (`2026.04.09`). This needs explicit explanation of the `_version()` chain in `scapy/__init__.py`.
- **Socket type clarification** — The default `L3PacketSocket` may not receive loopback ICMP replies correctly; the documentation should explain why `L3RawSocket` is needed for localhost testing and how `conf.L3socket` selection works.
- **Build pipeline context** — The user asks how Scapy "constructs the IP header by default when creating packets programmatically," which requires documenting the entire `build()` → `self_build()` → `post_build()` chain, not just the IP-specific code.
- **Test failure context** — Some test campaigns (cert.uts, sslv2.uts, tls.uts) fail due to missing TLS class definitions in the non-root environment. The documentation must explain these are TLS/crypto-specific failures, not core Scapy issues.

## 0.2 Documentation Discovery and Analysis

### 0.2.1 Existing Documentation Infrastructure Assessment

Repository analysis reveals a mature, Sphinx-based documentation ecosystem with ReadTheDocs hosting, alongside a comprehensive custom test framework (UTscapy) with `.uts` test files.

**Documentation Framework:** Sphinx `>=3.0.0` with `sphinx_rtd_theme>=0.4.3`
**Documentation Generator Configuration:** `doc/scapy/conf.py`
**API Documentation Tool:** `sphinx-apidoc` via the `apitree` tox environment
**Diagram Tools:** Mermaid (in tech spec), no built-in diagram tool in repo docs
**Hosting:** ReadTheDocs at `https://scapy.readthedocs.io`, configured via `.readthedocs.yml`

| Component | Location | Purpose |
|-----------|----------|---------|
| Sphinx source tree | `doc/scapy/` | Manual documentation (RST format) |
| Sphinx config | `doc/scapy/conf.py` | Build configuration with autodoc, napoleon, linkcode extensions |
| ReadTheDocs config | `.readthedocs.yml` | Build on Ubuntu 20.04 with Python 3.9, outputs HTML/EPUB/PDF |
| API tree generator | tox `apitree` environment | Generates API reference from docstrings via `sphinx-apidoc` |
| Docs build command | `sphinx-build -W --keep-going -b html . _build/html` | Strict warning-as-error build |
| Notebooks | `doc/notebooks/` | Interactive tutorials (Jupyter) |
| Vim syntax | `doc/syntax/` | Syntax highlighting support |

**Existing documentation files discovered:**

| File | Content |
|------|---------|
| `doc/scapy/introduction.rst` | Project overview, quick demo, philosophy |
| `doc/scapy/installation.rst` | Platform-specific installation guides |
| `doc/scapy/usage.rst` | Interactive tutorial (77KB — the largest doc file) |
| `doc/scapy/advanced_usage.rst` | Advanced features, automata, pipes |
| `doc/scapy/build_dissect.rst` | How to build new protocol dissectors |
| `doc/scapy/development.rst` | Development guide and contribution info |
| `doc/scapy/extending.rst` | Extending Scapy with custom layers |
| `doc/scapy/routing.rst` | Routing table interaction |
| `doc/scapy/troubleshooting.rst` | Common issues and solutions |
| `doc/scapy/functions.rst` | Functions reference |
| `doc/scapy/index.rst` | Documentation index / table of contents |
| `README.md` | Project overview, quick start, badges |
| `CONTRIBUTING.md` | Contributor guide |

### 0.2.2 Repository Code Analysis for Documentation

The following source code areas were examined to support the documentation requirements:

**Packet construction and IP header logic:**
- `scapy/packet.py` — Core `Packet` class with `build()` (line 746), `do_build()` (line 724), `self_build()` (line 678), `post_build()` (line 758), `show()` (line 1459), and `__truediv__` (line 596) for layer stacking
- `scapy/layers/inet.py` — `IP` class (line 521) with `fields_desc` (line 524) and `post_build()` (line 539); `ICMP` class (line 952) with its `fields_desc` and `post_build()` (line 979); `bind_layers(IP, ICMP, frag=0, proto=1)` binding
- `scapy/fields.py` — `IPField` (line 796), `SourceIPField` (line 854) with `__findaddr()` for route-based source IP resolution, `DestIPField` (line 503 of inet.py) inheriting from `DestField` (line 729 of fields.py)

**Interactive console:**
- `scapy/main.py` — `interact()` function (line 503) with banner/logo generation (lines 593–651), `QUOTES` list (line 47), `_scapy_builtins()` (line 302)
- `scapy/__init__.py` — `_version()` chain (line 122), `VERSION` assignment (line 168)

**Send/receive operations:**
- `scapy/sendrecv.py` — `send()` (line 422), `sr()` (line 635), `sr1()` (line 656)
- `scapy/config.py` — `conf.L3socket`, `conf.route`

**Test infrastructure:**
- `scapy/tools/UTscapy.py` — Custom test runner with `usage()` (line 851), campaign-based execution
- `test/configs/linux.utsc` — Linux test configuration with 193 `.uts` test files across `test/*.uts`, `test/scapy/layers/*.uts`, `test/contrib/*.uts`, and `test/tools/*.uts`
- Total test cases identified: 5,281 across 193 `.uts` files

### 0.2.3 Web Search Research Conducted

No external web searches were required for this documentation task. All answers are derived directly from the source code at commit `0925ada48540` and from live execution within the installed environment. The tech spec sections 1.1, 4.1, 4.2, 4.10, and 8.6 provided supplementary architectural context.

## 0.3 Documentation Scope Analysis

### 0.3.1 Code-to-Documentation Mapping

The documentation task requires creating a single comprehensive Markdown file that answers six specific questions. Each question maps to specific source modules that must be analyzed and cited.

**Module: `scapy/main.py` — Interactive Console Startup**
- Public APIs: `interact()`, `_scapy_builtins()`, `_read_config_file()`
- Current documentation: Exists in `doc/scapy/introduction.rst` and `doc/scapy/usage.rst`, but does not cover the specific startup banner details or the version display from a tagless commit
- Documentation needed: Exact startup output, banner composition logic, QUOTES list, version display behavior

**Module: `scapy/__init__.py` — Version Resolution**
- Public APIs: `_version()`, `_version_from_git_describe()`, `_version_from_git_archive()`, `_parse_tag()`, `VERSION`
- Current documentation: Not explicitly documented beyond the code docstrings
- Documentation needed: Explanation of the four-fallback version chain, behavior when no git tags exist

**Module: `scapy/packet.py` — Packet Build Pipeline**
- Public APIs: `Packet.build()`, `Packet.do_build()`, `Packet.self_build()`, `Packet.post_build()`, `Packet.show()`, `Packet.__truediv__()`
- Current documentation: Covered at high level in `doc/scapy/build_dissect.rst`
- Documentation needed: Step-by-step walkthrough with line-number citations for the specific case of `IP()/ICMP()` construction

**Module: `scapy/layers/inet.py` — IP and ICMP Classes**
- Public APIs: `IP` (fields_desc, post_build, route), `ICMP` (fields_desc, post_build), `bind_layers(IP, ICMP, ...)`
- Current documentation: Protocol-level docs in `doc/scapy/usage.rst`
- Documentation needed: Field-level detail showing defaults, None-valued auto-computed fields, and the exact post_build auto-population logic

**Module: `scapy/fields.py` — Field Types for IP**
- Public APIs: `SourceIPField`, `DestField`, `IPField`, `BitField`, `ByteEnumField`, `XShortField`
- Current documentation: API docstrings only
- Documentation needed: Explanation of `SourceIPField.__findaddr()` route-based resolution and `DestIPField.dst_from_pkt()` default destination logic

**Module: `scapy/sendrecv.py` — Packet Transmission**
- Public APIs: `send()`, `sr()`, `sr1()`
- Current documentation: Covered in `doc/scapy/usage.rst` interactive tutorial
- Documentation needed: Specific walkthrough of sending ICMP to localhost, including socket selection and stimulus-response matching

**Module: `scapy/tools/UTscapy.py` — Test Framework**
- Public APIs: `main()`, `run_campaign()`, `usage()`, `TestCampaign`
- Current documentation: `test/configs/README.md` for configuration format
- Documentation needed: Test suite execution output, campaign counts, pass/fail summary

### 0.3.2 Documentation Gap Analysis

Given the requirements and repository analysis, the documentation gaps that this task addresses include:

- **No existing Q&A document for this commit** — The `blitzy/documentation/` directory does not yet exist. A new file `scapy_0925ada48540.md` must be created from scratch.
- **Version behavior at tagless commits** — Existing documentation does not explain what happens when Scapy runs from a commit without git tags (fallback to file timestamp).
- **IP header auto-computation specifics** — While `build_dissect.rst` covers the general dissection framework, it does not walk through the specific case of `IP.post_build()` computing `ihl`, `len`, and `chksum`.
- **Localhost ICMP behavior** — No existing documentation covers the specific behavior of sending ICMP to `127.0.0.1` with `sr1()`, including the `L3PacketSocket` vs `L3RawSocket` distinction for loopback.
- **Full test suite summary** — No existing file in the repo contains a comprehensive pass/fail summary from the UTscapy test runner with the full Linux non-root configuration.

## 0.4 Documentation Implementation Design

### 0.4.1 Documentation Structure Planning

The output document follows a single-file Q&A format as required by the `SWE-AtlasQnA-Repo` rule. All content is consolidated into one Markdown file.

```
blitzy/
└── documentation/
    └── scapy_0925ada48540.md
        ├── Title and Introduction
        ├── Section 1: Scapy Shell Startup
        │   ├── Banner and ASCII Logo
        │   ├── Version Information
        │   └── Startup Internals (source walkthrough)
        ├── Section 2: ICMP Echo Request Construction
        │   ├── Creating IP()/ICMP()
        │   ├── show() Output (pre-build)
        │   └── Auto-Populated Fields (post-build)
        ├── Section 3: IP Header Construction Source Walkthrough
        │   ├── Build Pipeline: build() → do_build() → self_build() → post_build()
        │   ├── IP.post_build() (ihl, len, chksum computation)
        │   ├── SourceIPField route-based resolution
        │   ├── DestIPField default address logic
        │   └── bind_layers and proto field auto-set
        ├── Section 4: Sending ICMP to Localhost
        │   ├── sr1() execution and network-layer behavior
        │   ├── Echo-reply dissection
        │   └── Socket type considerations (L3PacketSocket vs L3RawSocket)
        ├── Section 5: Test Suite Execution
        │   ├── UTscapy framework overview
        │   ├── Test configuration and file counts
        │   └── Pass/fail summary
        └── References (source files cited)
```

### 0.4.2 Content Generation Strategy

**Information Extraction Approach:**
- Extract startup banner text from `scapy/main.py` lines 593–651 and reproduce the exact output
- Extract `IP.fields_desc` from `scapy/layers/inet.py` line 524 and `ICMP.fields_desc` from line 954
- Generate `show()` output by running `IP()/ICMP()` in the installed Scapy environment (Python 3.10.20)
- Extract the build pipeline logic from `scapy/packet.py` lines 678–758 and `scapy/layers/inet.py` lines 539–551
- Capture `sr1()` execution output from live loopback ICMP transmission
- Aggregate UTscapy test results from running with `-c test/configs/linux.utsc -N -b` flags

**Documentation Standards:**
- Markdown formatting with proper headers (`#`, `##`, `###`)
- Code examples using fenced code blocks with `python` syntax highlighting
- All technical claims include source citations in the format: `Source: scapy/file.py:LineNumber`
- Tables for structured data (field lists, test results)
- Consistent terminology aligned with Scapy's own naming conventions (e.g., "layer stacking," "post_build," "fields_desc")

### 0.4.3 Diagram and Visual Strategy

The documentation will include Mermaid diagrams for:

- **Packet build pipeline** — Flowchart tracing `build()` → `do_build()` → `self_build()` → `post_build()` with the IP-specific overrides highlighted
- **IP header auto-population** — Diagram showing which fields start as `None` and what values they receive during `post_build()`
- **ICMP send/receive flow** — Sequence diagram covering `sr1()` → socket → kernel → echo-reply → dissection

No screenshots or external images are required.

## 0.5 Documentation File Transformation Mapping

### 0.5.1 File-by-File Documentation Plan

| Target Documentation File | Transformation | Source Code/Docs | Content/Changes |
|---------------------------|----------------|------------------|-----------------|
| `blitzy/documentation/scapy_0925ada48540.md` | CREATE | `scapy/main.py`, `scapy/__init__.py`, `scapy/packet.py`, `scapy/fields.py`, `scapy/layers/inet.py`, `scapy/sendrecv.py`, `scapy/tools/UTscapy.py`, `test/configs/linux.utsc` | Comprehensive Q&A document answering all six user questions with source code analysis, live execution output, and rationale |

**Documentation Transformation Modes Applied:**
- **CREATE** — The file `blitzy/documentation/scapy_0925ada48540.md` does not exist and must be created from scratch
- No UPDATE, DELETE, or REFERENCE transformations are needed — no existing documentation files are modified

### 0.5.2 New Documentation File Detail

```
File: blitzy/documentation/scapy_0925ada48540.md
Type: Technical Q&A / Exploratory Reference Guide
Source Code:
  - scapy/__init__.py (version resolution)
  - scapy/main.py (interactive console startup)
  - scapy/packet.py (Packet base class, build pipeline)
  - scapy/fields.py (SourceIPField, DestField, IPField)
  - scapy/layers/inet.py (IP class, ICMP class, bind_layers)
  - scapy/sendrecv.py (send, sr, sr1)
  - scapy/config.py (conf object, L3socket)
  - scapy/tools/UTscapy.py (test runner)
  - test/configs/linux.utsc (test configuration)
Sections:
  - Introduction (commit context, environment setup)
  - Scapy Shell Startup (banner, version, QUOTES)
  - ICMP Echo Request Construction (fields_desc, show() output)
  - IP Header Auto-Population (post_build source walkthrough)
  - Sending ICMP to Localhost (sr1 execution, network behavior)
  - Test Suite Execution (UTscapy run, pass/fail summary)
  - References (all source files cited)
Diagrams:
  - Mermaid flowchart: Packet build pipeline
  - Mermaid diagram: IP field auto-computation
  - Mermaid sequence: ICMP send/receive via sr1()
Key Citations:
  - scapy/layers/inet.py:521-551 (IP class and post_build)
  - scapy/layers/inet.py:952-1010 (ICMP class and post_build)
  - scapy/packet.py:678-758 (build pipeline)
  - scapy/fields.py:854-896 (SourceIPField)
  - scapy/main.py:503-651 (interact and banner)
  - scapy/__init__.py:122-168 (version resolution)
  - scapy/sendrecv.py:635-665 (sr and sr1)
```

### 0.5.3 Documentation Configuration Updates

No documentation configuration files need to be modified. The new file is placed in a standalone `blitzy/documentation/` directory that is independent of the Sphinx documentation build system. Specifically:

- `doc/scapy/conf.py` — No changes needed (Sphinx config is unrelated to this output)
- `.readthedocs.yml` — No changes needed (this document is not part of the ReadTheDocs build)
- `tox.ini` — No changes needed (no new tox environments required)
- `pyproject.toml` — No changes needed

### 0.5.4 Cross-Documentation Dependencies

- The new document is self-contained and does not depend on or link to any existing documentation files
- No table of contents, navigation, or index updates are required
- No shared content or includes are used
- The document references source code by path and line number for traceability but does not create hyperlinks to the Sphinx-built documentation

## 0.6 Dependency Inventory

### 0.6.1 Documentation Dependencies

The following tools and packages are relevant to producing and verifying the documentation output:

| Registry | Package Name | Version | Purpose |
|----------|--------------|---------|---------|
| PyPI | scapy | 2026.04.09 (dev, commit 0925ada4) | The library under analysis, installed in editable mode from source |
| PyPI | setuptools | >=62.0.0 | Build backend for Scapy's `pyproject.toml` |
| System | Python | 3.10.20 | Runtime matching the highest explicitly documented supported version in pyproject.toml classifiers |
| PyPI | mock | 5.2.0 | Test dependency required by UTscapy regression tests |
| PyPI | cryptography | 46.0.7 | Test dependency for TLS/cert-related test campaigns |
| PyPI | ipython | 8.39.0 | Optional dependency for enhanced interactive shell |
| PyPI | coverage | 7.13.5 | Code coverage collection during test execution |

**Notes on version selection:**
- Python 3.10 was selected as the highest version explicitly listed in `pyproject.toml` classifiers (`Programming Language :: Python :: 3.10`). The `tox.ini` envlist includes `py311`, but the official classifiers are authoritative for declared support.
- Scapy itself has zero required runtime dependencies on Linux (`pyproject.toml` has no `dependencies` key), confirming the "zero core dependencies" design principle.
- The `mock`, `cryptography`, `ipython`, and `coverage` packages are test/development dependencies from the `[testenv]` section of `tox.ini` (line 29).

### 0.6.2 Documentation Reference Updates

No documentation reference or link updates are required. The new file is placed in a new directory (`blitzy/documentation/`) and does not modify or link to any existing documentation. There are no old-to-new link transformation rules to apply.

## 0.7 Coverage and Quality Targets

### 0.7.1 Documentation Coverage Metrics

The documentation task targets six specific questions. Coverage is measured against each question being fully answered with source evidence.

| Question | Source Modules Required | Coverage Target |
|----------|----------------------|-----------------|
| Scapy shell startup and banner | `scapy/main.py` (lines 503–651) | 100% — Full banner output with version |
| Version identification | `scapy/__init__.py` (lines 122–168) | 100% — All four fallback methods explained |
| ICMP echo request construction and `show()` | `scapy/layers/inet.py` (lines 521–538, 952–978) | 100% — Every IP and ICMP field documented |
| IP header auto-population source walkthrough | `scapy/packet.py` (lines 678–758), `scapy/layers/inet.py` (lines 539–551), `scapy/fields.py` (lines 854–896) | 100% — Full build pipeline traced |
| Sending ICMP to localhost | `scapy/sendrecv.py` (lines 635–665), `scapy/config.py` | 100% — Send, receive, and dissect documented |
| Test suite execution summary | `scapy/tools/UTscapy.py`, `test/configs/linux.utsc` | 100% — Counts, campaigns, and pass/fail reported |

**Current coverage status (pre-documentation):**
- User questions answered: 0/6 (0%)
- Target: 6/6 (100%)

### 0.7.2 Documentation Quality Criteria

**Completeness requirements:**
- Every user question receives a dedicated section with a direct answer
- All answers include the specific source file paths and line numbers as evidence
- Rationale and thinking are provided alongside raw outputs
- Auto-populated IP fields are shown both before and after the `build()` call to contrast `None` values with computed values

**Accuracy validation:**
- Code examples are captured from live execution in the Python 3.10.20 / Scapy 2026.04.09 environment
- The `show()` output is the exact text produced by the installed Scapy version at commit `0925ada48540`
- Test results are from an actual UTscapy run, not estimated or synthesized
- IP header field defaults match the `fields_desc` declaration in `scapy/layers/inet.py` line 524

**Clarity standards:**
- Each section begins with a direct answer to the question, followed by supporting evidence
- Technical concepts (e.g., `post_build`, `fields_desc`, layer stacking) are explained before being used
- Consistent terminology from Scapy's own documentation and docstrings is used throughout

**Maintainability:**
- Source citations use the format `Source: path/to/file.py:LineNumber` for traceability
- The document is tied to commit `0925ada48540` and clearly states this scope

### 0.7.3 Example and Diagram Requirements

- Minimum code examples: One per question (six total), plus additional intermediate examples for the build pipeline walkthrough
- Diagram types: Three Mermaid diagrams (build pipeline flowchart, IP field auto-computation, ICMP send/receive sequence)
- Code example verification: All examples were executed in the live environment and outputs captured verbatim
- Visual content freshness: Tied to commit `0925ada48540` — diagrams reflect this specific codebase version

## 0.8 Scope Boundaries

### 0.8.1 Exhaustively In Scope

**New documentation files:**
- `blitzy/documentation/scapy_0925ada48540.md` — The sole deliverable, a comprehensive Q&A document

**Source code to be analyzed (read-only):**
- `scapy/__init__.py` — Version resolution logic
- `scapy/main.py` — Interactive console startup, banner, QUOTES
- `scapy/packet.py` — Packet base class, build/show pipeline
- `scapy/fields.py` — SourceIPField, DestField, IPField, BitField, ByteEnumField
- `scapy/layers/inet.py` — IP class, ICMP class, IPerror, bind_layers
- `scapy/sendrecv.py` — send(), sr(), sr1() functions
- `scapy/config.py` — conf object, L3socket, route configuration
- `scapy/tools/UTscapy.py` — Test runner implementation
- `test/configs/linux.utsc` — Linux non-root test configuration
- `test/**/*.uts` — Test case files (counted, not individually documented)
- `pyproject.toml` — Package metadata, Python version constraints, optional dependencies
- `tox.ini` — Test environments, CI configuration
- `.readthedocs.yml` — Documentation hosting configuration

**Live execution tasks (temporary, cleaned up after):**
- Scapy shell startup capture
- `IP()/ICMP()` construction and `show()` output
- `sr1(IP(dst='127.0.0.1')/ICMP(), timeout=3)` transmission and response capture
- UTscapy test suite run with `linux.utsc` configuration, non-root mode

### 0.8.2 Explicitly Out of Scope

- **Source code modifications** — No existing files will be modified, per user instruction: "Don't modify any source files"
- **Test file modifications** — Test `.uts` files are read-only; no test changes
- **Feature additions or code refactoring** — This is a documentation-only task
- **Deployment configuration changes** — `.readthedocs.yml`, `tox.ini`, `pyproject.toml` are not modified
- **Sphinx documentation updates** — The `doc/scapy/*.rst` files are not modified; the output goes to `blitzy/documentation/`
- **IPv6 documentation** — Only IPv4 (`IP`) and ICMP are in scope per the user's questions
- **TLS/crypto layer documentation** — TLS test failures are noted but TLS internals are not documented
- **Windows or BSD platform behavior** — Only Linux behavior (PF_PACKET sockets, loopback interface) is documented
- **Contrib module documentation** — The `scapy/contrib/` protocols are not in scope
- **Root-mode test execution** — Only non-root (`-N` flag) tests are executed
- **Unrelated documentation** — No documentation beyond what the user explicitly requested

## 0.9 Execution Parameters

### 0.9.1 Documentation-Specific Instructions

| Parameter | Value |
|-----------|-------|
| **Documentation build command** | N/A — output is a standalone Markdown file, not a Sphinx project |
| **Documentation preview command** | Any Markdown viewer or `cat blitzy/documentation/scapy_0925ada48540.md` |
| **Diagram generation** | Mermaid diagrams embedded inline in Markdown (rendered by any Mermaid-compatible viewer) |
| **Documentation deployment** | N/A — file is committed to the repository at `blitzy/documentation/` |
| **Default format** | Markdown with Mermaid diagrams |
| **Citation requirement** | Every section must reference source files with `Source: path:line` format |
| **Style guide** | Q&A format: direct answer first, then rationale and evidence |
| **Validation** | Manual review; all code outputs verified against live execution |

### 0.9.2 Environment Configuration Used

| Component | Value |
|-----------|-------|
| **Python version** | 3.10.20 |
| **Virtual environment** | `/tmp/scapy_venv` |
| **Scapy installation** | Editable install from `/tmp/blitzy/scapy/scapy_0925ada48540_ca08b7` |
| **Scapy version reported** | `2026.04.09` (fallback to file modification date, no git tags) |
| **Commit hash** | `0925ada485406684174d6f068dbd85c4154657b3` |
| **Branch name** | `scapy_0925ada48540` |
| **Test runner** | `python -m scapy.tools.UTscapy` |
| **Test config** | `test/configs/linux.utsc` with `-N` (non-root) and `-b` (don't break on failure) |

### 0.9.3 Live Execution Outputs Captured

The following live outputs were captured during context gathering and will be included in the documentation:

- **Scapy startup banner** — ASCII logo (19 lines), version line, GitHub URL, welcome message, random quote
- **`IP()/ICMP()` repr** — `<IP  frag=0 proto=icmp |<ICMP  |>>`
- **`IP()/ICMP()` show() pre-build** — 16 IP fields + 6 ICMP fields, including `ihl=None`, `len=None`, `chksum=None`
- **`IP()/ICMP()` show() post-build** — Same fields with `ihl=5`, `len=28`, `chksum=0x7cde`, `src=127.0.0.1`, `dst=127.0.0.1`
- **`sr1()` ICMP echo-reply** — `<IP version=4 ihl=5 tos=0x0 len=28 id=49063 ... proto=icmp chksum=0xbd37 src=127.0.0.1 dst=127.0.0.1 |<ICMP type=echo-reply code=0 chksum=0x0 ...>>`
- **UTscapy test summary** — 190 campaigns loaded, 4,760 tests passed, 257 tests failed (primarily TLS/cert-related), across 5,017 executed test cases

## 0.10 Rules for Documentation

The following rules are explicitly enforced by the user's instructions and the implementation rule `SWE-AtlasQnA-Repo`:

- **Do not modify any existing files in the source repository** — All analysis is read-only. The only file created is the new document at `blitzy/documentation/scapy_0925ada48540.md`.
- **Do not make assumptions, base your answers on the code as the truth** — Every claim must be traceable to a specific source file and line number. No speculative or generic answers.
- **Provide thinking / rationale behind the answers** — Each section must explain *why* the answer is what it is, not just *what* the answer is. Include the reasoning process.
- **Create a new markdown document named `<source_branch_name>.md`** — The branch name is `scapy_0925ada48540`, so the file is `scapy_0925ada48540.md`.
- **Place the generated document in the `blitzy/documentation` directory** — The directory must be created if it does not exist.
- **Clean up temporary scripts after use** — If any temporary Python scripts are created for exploration, they must be removed before completion.
- **No source code modification under any circumstances** — Reinforced by both the user ("Don't modify any source files") and the implementation rule.

## 0.11 References

### 0.11.1 Source Files and Folders Searched

The following files were directly read and analyzed to derive the conclusions in this document:

| File Path | Purpose of Inspection |
|-----------|----------------------|
| `scapy/__init__.py` | Version resolution logic (`_version()`, `_version_from_git_describe()`, `_version_from_git_archive()`, `_parse_tag()`, `VERSION`) |
| `scapy/main.py` | Interactive console startup (`interact()`), banner composition (lines 593–651), `QUOTES` list (line 47), `_scapy_builtins()` |
| `scapy/packet.py` | Core Packet class: `__init__()` (line 141), `build()` (line 746), `do_build()` (line 724), `self_build()` (line 678), `post_build()` (line 758), `show()` (line 1459), `__truediv__()` (line 596) |
| `scapy/fields.py` | Field types: `IPField` (line 796), `SourceIPField` (line 854) with `__findaddr()`, `DestField` (line 729) |
| `scapy/layers/inet.py` | `IP` class (line 521) with `fields_desc` and `post_build()` (line 539); `ICMP` class (line 952) with `post_build()` (line 979); `DestIPField` (line 503); `bind_layers()` calls |
| `scapy/sendrecv.py` | `send()` (line 422), `sr()` (line 635), `sr1()` (line 656) |
| `scapy/config.py` | `conf` object, `L3socket` configuration |
| `scapy/tools/UTscapy.py` | Test runner: `usage()` (line 851), `run_campaign()`, PASSED/FAILED reporting (line 619) |
| `test/configs/linux.utsc` | Linux non-root test configuration (testfiles, preexec, kw_ko) |
| `pyproject.toml` | Package metadata, Python version constraints (`>=3.7, <4`), optional dependencies, build system |
| `tox.ini` | Test environments, CI configuration, Python version matrix (py27–py311) |
| `.readthedocs.yml` | ReadTheDocs configuration (Ubuntu 20.04, Python 3.9, EPUB/PDF outputs) |
| `README.md` | Project overview, quick start, badges, platform support |
| `CONTRIBUTING.md` | Contributor guide |
| `doc/scapy/conf.py` | Sphinx build configuration (extensions, theme, autodoc settings) |
| `doc/scapy/*.rst` | Existing RST documentation files (introduction, usage, build_dissect, etc.) |

**Folders explored:**

| Folder Path | Purpose of Inspection |
|-------------|----------------------|
| `/` (repository root) | Top-level structure, all immediate children |
| `scapy/` | Core package structure, module listing |
| `scapy/layers/` | Protocol layer implementations |
| `scapy/tools/` | UTscapy test runner |
| `test/` | Test files, configurations, contrib tests |
| `test/configs/` | Test configuration files (.utsc) |
| `doc/` | Documentation ecosystem |
| `doc/scapy/` | Sphinx manual source tree |

### 0.11.2 Tech Spec Sections Referenced

| Section | Content Used |
|---------|-------------|
| 1.1 Executive Summary | Project overview, version info, stakeholders, value proposition |
| 4.1 High-Level System Workflow | End-to-end processing architecture, feature dependency flow |
| 4.2 Core Packet Engine Processes | Build lifecycle, dissection lifecycle, layer stacking |
| 4.10 Interactive Console Startup | Console initialization sequence, session persistence |
| 8.6 Documentation Infrastructure | ReadTheDocs config, Sphinx build, API doc generation |

### 0.11.3 Attachments and External URLs

No attachments were provided for this project. No Figma URLs were specified. No external URLs require documentation beyond the repository's own references to:
- `https://scapy.readthedocs.io` (official documentation)
- `https://github.com/secdev/scapy` (source repository)

