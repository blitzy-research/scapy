# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

### 0.1.1 Core Documentation Objective

Based on the provided requirements, the Blitzy platform understands that the documentation objective is to **create new documentation** that delivers a rigorous, runtime-verified investigation of Scapy's TCP stream reconstruction behavior when processing overlapping or retransmitted TCP segments. The documentation type is a **technical investigation and Q&A document** — a single Markdown file (`blitzy/documentation/scapy_0925ada48540.md`) placed in the destination repository.

The request is categorized as: **Create new documentation**
The documentation type is: **Technical investigation / Q&A document**

The documentation requirements, restated with enhanced clarity, are:

- **Reproduce a minimal TCP overlap flow** — Craft an application payload split across multiple TCP segments where at least one segment overlaps (retransmits) a byte range already covered by a prior segment. Two variants are required: one with identical bytes in the overlap and one with different bytes.
- **End-to-end offline reassembly path** — The demonstration must exercise Scapy's normal offline reassembly pipeline: write a synthetic pcap to disk with `wrpcap()`, read it back with `sniff(offline=..., session=TCPSession)`, and observe the reassembled output.
- **Built-in layers only** — Use only Scapy layers that already implement `tcp_reassemble` (such as `HTTP` via `load_layer("http")`). No new `Packet` subclasses, no `bind_layers()` calls, no direct import or instantiation of internal helpers like `StringBuffer`.
- **Concrete runtime evidence** — Provide verbatim terminal excerpts showing: (a) the exact command executed, (b) the reconstructed byte stream representation for each variant, (c) the exact byte length, and (d) which bytes "win" in the overlap region.
- **Code-path tracing** — Connect the observed runtime behavior back to the specific source files, functions, and lines in the Scapy codebase that implement the overlap resolution policy.
- **No source modifications** — Do not modify any repository source files. Temporary scripts and pcap files are permitted but must be cleaned up, with terminal evidence confirming their removal.

These documentation requirements translate to the following technical documentation strategy:

- To **document the overlap reproduction**, we will **create** `blitzy/documentation/scapy_0925ada48540.md` containing the complete investigation with annotated code, terminal output, and analysis.
- To **demonstrate the reassembly path**, we will write a self-contained Python script that uses `Ether()/IP()/TCP()/Raw()` packet construction, `wrpcap()` for pcap serialization, and `sniff(offline=..., session=TCPSession)` for reassembly — then capture its output verbatim.
- To **trace the code path**, we will walk through the call chain from `sniff()` → `TCPSession._process_packet()` → `StringBuffer.append()` → the `memoryview` overwrite on line 193 of `scapy/sessions.py`, explaining why the **last-writer-wins** policy produces the exact bytes observed.

### 0.1.2 Special Instructions and Constraints

- **No source file modifications**: The user explicitly states "Don't modify any repository source files." This is reinforced by the implementation rule: "Do not modify any existing files in the source repository."
- **Temporary artifact cleanup**: Temporary scripts and pcap files are allowed but must be removed after the run, with a brief terminal excerpt proving they are gone.
- **Built-in layer restriction**: Only built-in Scapy layers that already support TCP reassembly may be used. The following are prohibited: defining new `Packet` classes, calling `bind_layers()`, importing or instantiating `StringBuffer` directly.
- **Verbatim terminal excerpts**: The document must include exact, copy-paste-ready terminal output — not paraphrased descriptions.
- **Implementation rule**: The output must be a new Markdown document named `<source_branch_name>.md` (i.e., `scapy_0925ada48540.md`) placed in the `blitzy/documentation` directory. It must provide thinking/rationale behind the answers and base all answers on the code as truth.

### 0.1.3 Technical Interpretation

These documentation requirements translate to the following technical documentation strategy:

- To document the **overlap policy**, we will create a comprehensive Markdown document that begins with the question being investigated, walks through the experimental setup, presents the verbatim runtime evidence, and concludes with a source-code-level explanation of the observed behavior.
- To demonstrate the **identical-byte overlap**, we will construct a pcap where the server response (`HTTP/1.0 200 OK\r\nContent-Length: 4\r\n\r\nABCD`, 42 bytes) is split across three TCP segments, with segment 2 retransmitting positions [10:20) carrying the same bytes as segment 1. The expected outcome: 42 bytes correctly reassembled; the overlap region is overwritten with identical data (no net change).
- To demonstrate the **different-byte overlap**, we will construct the same pcap but with segment 2's overlap region replaced by `ZZZZZZZZZZ`. The expected outcome: the `memoryview` overwrite replaces positions [10:20) with the Z-bytes, corrupting the HTTP headers. This causes `HTTP.tcp_reassemble()` to return a partial packet early (30 bytes), and the third segment starts a new reassembly buffer (12 bytes).
- To trace the **code path**, we will document the call chain: `sniff()` (`scapy/sendrecv.py:1308`) → `AsyncSniffer._run()` → `PcapReader` (`scapy/utils.py:1367`) → `session.on_packet_received()` → `TCPSession._process_packet()` (`scapy/sessions.py:286`) → `StringBuffer.append()` (`scapy/sessions.py:179`) → `memoryview(self.content)[seq:seq+data_len] = data` (`scapy/sessions.py:193`).

### 0.1.4 Inferred Documentation Needs

Based on code analysis and the investigation:

- **`StringBuffer` overlap policy is undocumented**: The class at `scapy/sessions.py:161` contains XXX comments (lines 189–192, 199) indicating incomplete gap-tracking logic but no documentation of the overlap/overwrite behavior.
- **`TCPSession` reassembly interacts with application-layer parsing**: The overlap behavior has downstream effects on `tcp_reassemble()` implementations. When overlapping bytes corrupt protocol headers (e.g., HTTP), the application-layer parser may return partial results, fragmenting what should be a single reassembled stream.
- **The `data.full()` method always returns `True`** (line 199: `return True  # XXX`), meaning incomplete/missing segment detection is not yet implemented. This is relevant context for understanding why reassembly proceeds even with gaps or conflicts.
- **No existing documentation covers this behavior**: The existing `doc/scapy/layers/tcp.rst` file (67 lines) covers `StreamSocket`, `TCP_client` automaton, and external projects, but does not mention `TCPSession`, `StringBuffer`, or overlap handling. The `doc/scapy/usage.rst` file documents `TCPSession` usage but only from a how-to perspective, not the underlying overlap policy.

## 0.2 Documentation Discovery and Analysis

### 0.2.1 Existing Documentation Infrastructure Assessment

Repository analysis reveals a **Sphinx-based documentation system** with mature infrastructure but incomplete coverage of TCP reassembly internals.

**Documentation framework**: Sphinx (version `>=3.0.0`, declared in `pyproject.toml` docs extras)

**Documentation generator configuration**: `doc/scapy/conf.py` — configures Sphinx with autodoc, napoleon, todo, linkcode extensions; imports version metadata from the `scapy` package; supports HTML, LaTeX, man page, and Texinfo output.

**Documentation hosting**: ReadTheDocs at `https://scapy.readthedocs.io`, configured via `.readthedocs.yml` (Ubuntu 20.04, Python 3.9, pip install with `[docs]` extras, EPUB+PDF output enabled).

**Existing documentation tree** (from `doc/scapy/`):

| File | Coverage Area | Relevance to Task |
|------|--------------|-------------------|
| `doc/scapy/usage.rst` | Interactive usage, sniffing, sessions | HIGH — contains TCPSession usage docs (lines 780–830) |
| `doc/scapy/layers/tcp.rst` | TCP layer documentation | HIGH — 67 lines covering StreamSocket and TCP automaton only |
| `doc/scapy/layers/http.rst` | HTTP layer documentation | MEDIUM — documents HTTP usage but not tcp_reassemble internals |
| `doc/scapy/advanced_usage.rst` | ASN.1, Automata, PipeTools | LOW — does not cover sessions or reassembly |
| `doc/scapy/build_dissect.rst` | Packet layer definitions | LOW — about building new protocol layers |
| `doc/scapy/index.rst` | Master toctree | LOW — navigation only |

**API documentation tools**: `sphinx-apidoc` (via `tox` `apitree` environment) for auto-generating API docs from docstrings.

**Diagram tools**: Mermaid (used in tech spec sections). The existing Sphinx docs use SVG/PNG graphics in `doc/scapy/graphics/` but no Mermaid integration.

**Key finding — documentation gap**: The existing `doc/scapy/layers/tcp.rst` file does not mention `TCPSession`, `StringBuffer`, overlap handling, or the `tcp_reassemble` protocol. The `usage.rst` file covers TCPSession usage (lines 788–830) but only explains the `tcp_reassemble` API contract — not the underlying buffer management or overlap resolution.

### 0.2.2 Repository Code Analysis for Documentation

Search patterns used for code analysis:

- **TCP reassembly core**: `scapy/sessions.py` — `StringBuffer` (line 161), `TCPSession` (line 223), `_process_packet()` (line 286)
- **Built-in tcp_reassemble implementations**: Found in 7 locations across the codebase:
  - `scapy/layers/http.py:580` — HTTP 1.0/1.1 reassembly
  - `scapy/layers/dcerpc.py:537` — DCE/RPC reassembly
  - `scapy/layers/kerberos.py:1486` — Kerberos reassembly
  - `scapy/layers/tls/session.py:1087` — TLS session reassembly
  - `scapy/contrib/postgres.py:167,783,793` — PostgreSQL protocol reassembly
- **Sniff pipeline**: `scapy/sendrecv.py` — `sniff()` (line 1308), `AsyncSniffer._run()` (line 981)
- **PCAP I/O**: `scapy/utils.py` — `wrpcap()` (line 1095), `rdpcap()` (line 1136), `PcapReader` (line 1367)
- **Layer bindings for HTTP**: `scapy/layers/http.py:750` — `bind_layers(TCP, HTTP, sport=80, dport=80)` (creates separate sport=80 and dport=80 entries in `TCP.payload_guess`)

Key directories examined:
- `scapy/sessions.py` — Core reassembly logic
- `scapy/sendrecv.py` — Sniff/send pipeline
- `scapy/utils.py` — PCAP file I/O
- `scapy/layers/http.py` — HTTP layer with tcp_reassemble
- `scapy/layers/inet.py` — IP/TCP layer definitions
- `doc/scapy/` — Existing documentation source tree
- `doc/scapy/layers/` — Protocol-specific documentation

### 0.2.3 Web Search Research Conducted

No external web search was required for this task. All findings are derived directly from the Scapy source code in the repository at commit `0925ada4` (branch `scapy_0925ada48540`). The investigation is entirely code-driven, consistent with the user's directive to base answers on the code as truth rather than theoretical explanations.

## 0.3 Documentation Scope Analysis

### 0.3.1 Code-to-Documentation Mapping

The following modules require documentation analysis to produce the output document:

- **Module: `scapy/sessions.py`**
  - Public APIs relevant: `StringBuffer` (class, line 161), `StringBuffer.append(data, seq)` (line 179), `TCPSession` (class, line 223), `TCPSession._process_packet(pkt)` (line 286)
  - Current documentation: Usage-level docs exist in `doc/scapy/usage.rst` (lines 788–830). No documentation of internal overlap resolution policy.
  - Documentation needed: Detailed explanation of `StringBuffer.append()` overwrite semantics, runtime evidence of last-writer-wins behavior, annotated call-path walkthrough.

- **Module: `scapy/layers/http.py`**
  - Public APIs relevant: `HTTP.tcp_reassemble(cls, data, metadata, _)` (line 580), `HTTP.guess_payload_class()` (line 637), `bind_layers(TCP, HTTP, sport=80, dport=80)` (line 750)
  - Current documentation: `doc/scapy/layers/http.rst` covers HTTP usage. No documentation of how `tcp_reassemble` interacts with corrupted reassembled data.
  - Documentation needed: Explanation of how HTTP's tcp_reassemble responds to corrupted headers caused by different-byte overlaps, including the early-return path when `isinstance(http_packet.payload, _HTTPContent)` is `False`.

- **Module: `scapy/sendrecv.py`**
  - Public APIs relevant: `sniff()` (line 1308), `AsyncSniffer._run()` (line 981), offline pcap reading path (lines 1107–1143)
  - Current documentation: `doc/scapy/usage.rst` covers sniff() usage extensively.
  - Documentation needed: Brief note on the call path from `sniff(offline=..., session=TCPSession)` through `PcapReader` to `session.on_packet_received()`.

- **Module: `scapy/utils.py`**
  - Public APIs relevant: `wrpcap()` (line 1095), `PcapReader` (line 1367)
  - Current documentation: Covered in usage docs.
  - Documentation needed: Only as a reference in the call-path trace (pcap write/read).

### 0.3.2 Documentation Gap Analysis

Given the requirements and repository analysis, the critical documentation gaps are:

- **Undocumented overlap policy**: `StringBuffer.append()` at `scapy/sessions.py:179-193` implements an unconditional `memoryview` overwrite with no conflict detection. This is the core behavior the user is asking about, and it is completely absent from all existing documentation.
- **No runtime demonstration of reassembly edge cases**: The existing docs (`usage.rst`) show only happy-path TCPSession usage. No examples demonstrate behavior with overlapping segments.
- **XXX/TODO markers in StringBuffer**: Lines 189–192 and 199 of `sessions.py` contain explicit `# XXX` comments indicating incomplete gap-tracking and incomplete `full()` logic. These are not documented anywhere.
- **Missing interaction documentation**: The effect of StringBuffer overlap policy on downstream `tcp_reassemble` implementations (e.g., HTTP header corruption causing early return) is not documented.
- **No `blitzy/documentation/` directory exists**: The output directory must be created as part of this task.

## 0.4 Documentation Implementation Design

### 0.4.1 Documentation Structure Planning

The output is a single Markdown document following the implementation rule structure:

```
blitzy/
└── documentation/
    └── scapy_0925ada48540.md
```

The document structure within `scapy_0925ada48540.md`:

```
# TCP Stream Reconstruction with Overlapping Segments in Scapy

#### Question Summary

  (Restates the investigation question)

#### Approach and Rationale

  (Explains the experimental design and why these specific choices)

#### Experimental Setup

#### Test Script

    (Annotated Python script — read-only against repo)
#### Pcap Construction

    (Describes the two pcap variants)

#### Runtime Evidence

#### Variant A: Identical-Byte Overlap

    (Verbatim terminal excerpt, bytes, length, winner)
#### Variant B: Different-Byte Overlap

    (Verbatim terminal excerpt, bytes, length, winner)
#### Cleanup Verification

    (Terminal excerpt showing temp files removed)

#### Code-Path Analysis

#### Call Chain Overview

    (Mermaid diagram of the call path)
## StringBuffer.append() — The Overlap Policy

    (Annotated code walkthrough, line references)
## HTTP.tcp_reassemble() Interaction

    (How corrupted headers cause early return)

#### Key Findings Summary

  (Concise answers to each part of the question)
```

### 0.4.2 Content Generation Strategy

**Information Extraction Approach**:

- Extract the overlap resolution logic from `scapy/sessions.py:179-193` by analyzing the `StringBuffer.append()` method.
- Generate runtime evidence by executing a self-contained Python script that uses only built-in Scapy APIs (`wrpcap`, `sniff`, `TCPSession`, `load_layer("http")`).
- Create the call-path diagram by tracing from `sniff()` in `scapy/sendrecv.py:1308` through `TCPSession._process_packet()` in `scapy/sessions.py:286` to `StringBuffer.append()` in `scapy/sessions.py:179`.

**Template Application**: The document follows the implementation rule format: a comprehensive Markdown Q&A with thinking/rationale, grounded entirely in the codebase.

**Documentation Standards**:

- Markdown formatting with proper headers (`# ## ###`)
- Mermaid diagram for the reassembly call chain
- Code blocks with Python syntax highlighting for the test script
- Terminal output in fenced code blocks (no language tag, for raw output)
- Source citations as inline references: `Source: scapy/sessions.py:193`
- Tables for structured comparison of variant outcomes

### 0.4.3 Diagram and Visual Strategy

Mermaid diagrams to create within `scapy_0925ada48540.md`:

- **Reassembly call-chain diagram**: Sequence diagram showing `sniff()` → `PcapReader` → `TCPSession.on_packet_received()` → `_ip_process_packet()` → `_process_packet()` → `StringBuffer.append()` → `memoryview overwrite` → `tcp_reassemble()`.
- **StringBuffer state diagram**: Flowchart showing how `append()` handles the overlap region for both identical and different byte scenarios, with the unconditional `memoryview` overwrite at the decision point.

No screenshots or images are required. All visuals are text-based (Mermaid or ASCII).

## 0.5 Documentation File Transformation Mapping

### 0.5.1 File-by-File Documentation Plan

| Target Documentation File | Transformation | Source Code/Docs | Content/Changes |
|---------------------------|----------------|------------------|-----------------|
| `blitzy/documentation/scapy_0925ada48540.md` | CREATE | `scapy/sessions.py`, `scapy/layers/http.py`, `scapy/sendrecv.py`, `scapy/utils.py` | Complete technical investigation document: question restatement, experimental design rationale, annotated test script, verbatim terminal output for both overlap variants, code-path analysis with file/line references, Mermaid call-chain diagram, key findings summary |

**Transformation Modes Used**:

- **CREATE** — `blitzy/documentation/scapy_0925ada48540.md` — This is the sole deliverable. The directory `blitzy/documentation/` does not exist and must be created.
- No **UPDATE**, **DELETE**, or **REFERENCE** transformations are needed. No existing files are modified per the user's explicit constraint ("Don't modify any repository source files") and the implementation rule ("Do not modify any existing files in the source repository").

### 0.5.2 New Documentation File Detail

```
File: blitzy/documentation/scapy_0925ada48540.md
Type: Technical Investigation / Q&A Document
Source Code:
  - scapy/sessions.py (StringBuffer.append lines 179-193, TCPSession._process_packet lines 286-380)
  - scapy/layers/http.py (HTTP.tcp_reassemble lines 580-636, bind_layers line 750)
  - scapy/sendrecv.py (sniff() line 1308, AsyncSniffer._run lines 981-1306)
  - scapy/utils.py (wrpcap line 1095, PcapReader line 1367)
Sections:
  - Question Summary (restates the user's investigation question)
  - Approach and Rationale (experimental design, why HTTP on port 80, why three segments)
  - Experimental Setup (annotated Python script, pcap construction details)
  - Runtime Evidence — Variant A: Identical-Byte Overlap
    - Verbatim terminal excerpt with command and output
    - Reconstructed bytes: b'HTTP/1.0 200 OK\r\nContent-Length: 4\r\n\r\nABCD' (42 bytes)
    - Overlap region [10:20): b'00 OK\r\nCon' — same from both segments
    - Winner: seg2 (last writer), net effect: no change
  - Runtime Evidence — Variant B: Different-Byte Overlap
    - Verbatim terminal excerpt with command and output
    - Reconstructed bytes: b'HTTP/1.0 2ZZZZZZZZZZtent-Lengt' (30 bytes, first packet)
    - Second packet: b'h: 4\r\n\r\nABCD' (12 bytes, separate reassembly)
    - Overlap region [10:20): b'ZZZZZZZZZZ' — seg2 bytes overwrote seg1
    - Winner: seg2 (last writer), net effect: HTTP header corruption causing split output
  - Cleanup Verification (terminal excerpt showing temp files removed)
  - Code-Path Analysis (Mermaid diagram, annotated StringBuffer.append walkthrough)
  - HTTP.tcp_reassemble Interaction (how corrupted headers cause early return)
  - Key Findings Summary (concise answers)
Diagrams:
  - Mermaid sequence diagram: reassembly call chain from sniff() to memoryview overwrite
  - Mermaid flowchart: StringBuffer.append() overlap handling logic
Key Citations:
  - scapy/sessions.py:193 (memoryview overwrite — the policy)
  - scapy/sessions.py:179-188 (buffer extension logic)
  - scapy/sessions.py:286-380 (_process_packet full method)
  - scapy/sessions.py:199 (data.full() always returns True — XXX)
  - scapy/layers/http.py:580-636 (HTTP.tcp_reassemble)
  - scapy/layers/http.py:637-670 (HTTP.guess_payload_class)
  - scapy/sendrecv.py:1308-1312 (sniff entry point)
  - scapy/sendrecv.py:1107-1123 (offline pcap reading)
```

### 0.5.3 Documentation Configuration Updates

No documentation configuration files need to be created or modified. The output document is a standalone Markdown file that does not integrate into the Sphinx documentation build system. It resides in `blitzy/documentation/`, which is outside the `doc/scapy/` Sphinx source tree.

### 0.5.4 Cross-Documentation Dependencies

- **No cross-documentation dependencies**: The output document is self-contained. It references source code files by path and line number but does not depend on or link to any existing documentation files.
- **No navigation updates**: No `toctree`, `mkdocs.yml`, or sidebar configuration changes are required.
- **No index/glossary updates**: The document is standalone and does not contribute to any shared index.

## 0.6 Dependency Inventory

### 0.6.1 Documentation Dependencies

The following dependencies are relevant to this documentation exercise. All are already present in the repository or the Python standard library — no new packages need to be installed.

| Registry | Package Name | Version | Purpose |
|----------|-------------|---------|---------|
| PyPI | scapy | 2.7.0 (commit 0925ada4) | The project under investigation — provides `TCPSession`, `StringBuffer`, `sniff`, `wrpcap`, and all TCP/HTTP layer machinery |
| stdlib | python | >=3.7, <4 (3.12.3 used in test) | Runtime for the test script; `pyproject.toml` declares `requires-python = ">=3.7, <4"` |
| PyPI (optional) | sphinx | >=3.0.0 | Existing documentation build tool (declared in `pyproject.toml` docs extras); not required for this task |
| PyPI (optional) | sphinx_rtd_theme | >=0.4.3 | Existing documentation theme (declared in `pyproject.toml` docs extras); not required for this task |

**Key note**: The test script used for runtime evidence uses **only Scapy itself and Python standard library modules** (`sys`, `os`). No additional dependencies are introduced. The Scapy HTTP layer is loaded at runtime via `load_layer("http")`, which is a built-in Scapy function that loads from `scapy/layers/http.py` — no external package required.

### 0.6.2 Documentation Reference Updates

No documentation link updates are required. The new file `blitzy/documentation/scapy_0925ada48540.md` is a standalone document that does not participate in any existing link graph or navigation structure.

## 0.7 Coverage and Quality Targets

### 0.7.1 Documentation Coverage Metrics

**Current coverage analysis** (relevant to the user's question):

- TCP reassembly overlap policy documented: 0% — `StringBuffer.append()` overwrite semantics are completely undocumented in existing docs
- Runtime evidence of overlap behavior documented: 0% — no existing document demonstrates overlap handling with terminal output
- Code-path trace from `sniff()` to overlap resolution documented: 0% — existing `usage.rst` shows API usage but not internal call paths

**Target coverage** after this documentation task: 100% of the user's question is answered with:

- Both overlap variants (identical and different bytes) demonstrated end-to-end
- Verbatim terminal output for command, bytes, length, and winner for each variant
- Complete call-path trace from `sniff()` through `StringBuffer.append()` to the `memoryview` overwrite
- Explanation of why the observed bytes are produced, connecting runtime evidence to specific source lines

### 0.7.2 Documentation Quality Criteria

**Completeness requirements**:

- The document answers every sub-question in the user's prompt: overlap reproduction, two variants, exact bytes, byte length, which bytes win, code-path trace, file/function identification, cleanup verification
- All claims are supported by either runtime output or source code citations with file paths and line numbers
- The test script is fully self-contained and reproducible using only the repository checkout and `python3`

**Accuracy validation**:

- All terminal output included in the document is captured from actual execution against commit `0925ada4`
- All source code line numbers are verified against the repository at the current HEAD
- The overlap policy characterization ("last-writer-wins via unconditional `memoryview` assignment") is directly evidenced by `scapy/sessions.py:193`

**Clarity standards**:

- Technical accuracy with accessible explanations for each step of the call path
- Progressive disclosure: question → approach → evidence → analysis → findings
- Consistent use of "seg1", "seg2", "seg3" terminology and byte-position notation (e.g., `[10:20)`)

**Maintainability**:

- All source references include file path and line number (e.g., `scapy/sessions.py:193`)
- The document is tied to a specific commit (`0925ada4`) to prevent staleness confusion

### 0.7.3 Example and Diagram Requirements

- **Minimum examples**: Two complete runtime examples (Variant A: identical overlap, Variant B: different overlap), each with full terminal output
- **Diagram types**: Mermaid sequence diagram (call chain) and Mermaid flowchart (StringBuffer.append logic)
- **Code example verification**: The test script was executed at runtime and produced the terminal output included in the document; no synthetic or hypothetical output is used
- **Visual content freshness**: Tied to commit `0925ada4`; diagrams reflect the code as of that commit

## 0.8 Scope Boundaries

### 0.8.1 Exhaustively In Scope

**New documentation files**:

- `blitzy/documentation/scapy_0925ada48540.md` — The sole deliverable; comprehensive Q&A investigation document

**Source code files analyzed** (read-only, for documentation content extraction):

- `scapy/sessions.py` — `StringBuffer` class (line 161), `StringBuffer.append()` (line 179), `TCPSession` class (line 223), `TCPSession._process_packet()` (line 286), `TCPSession.on_packet_received()` (line 382)
- `scapy/layers/http.py` — `HTTP.tcp_reassemble()` (line 580), `HTTP.guess_payload_class()` (line 637), `bind_layers(TCP, HTTP, ...)` (line 750)
- `scapy/sendrecv.py` — `sniff()` (line 1308), `AsyncSniffer._run()` (line 981), offline pcap handling (lines 1107–1143)
- `scapy/utils.py` — `wrpcap()` (line 1095), `PcapReader` class (line 1367)
- `scapy/layers/inet.py` — `TCP` class (line 753) and `IP` class for packet construction context

**Existing documentation files examined** (read-only, for gap analysis):

- `doc/scapy/layers/tcp.rst` — Current TCP documentation (67 lines, no TCPSession coverage)
- `doc/scapy/usage.rst` — TCPSession usage documentation (lines 780–830)
- `doc/scapy/conf.py` — Sphinx configuration
- `.readthedocs.yml` — Documentation hosting configuration

**Runtime artifacts** (temporary, created and destroyed during investigation):

- `/tmp/_overlap_identical.pcap` — Pcap with identical-byte overlap (deleted after test)
- `/tmp/_overlap_different.pcap` — Pcap with different-byte overlap (deleted after test)
- `/tmp/tcp_overlap_test.py` — Test script (deleted after test)

### 0.8.2 Explicitly Out of Scope

- **Source code modifications**: No changes to any file under `scapy/`, `doc/`, `test/`, or any other repository directory. The user explicitly states: "Don't modify any repository source files." The implementation rule reinforces: "Do not modify any existing files in the source repository."
- **Test file modifications**: No changes to any test files.
- **Feature additions or code refactoring**: No changes to `StringBuffer.append()` logic, no addition of conflict detection, no fixes to the `XXX` markers in `sessions.py`.
- **Existing documentation updates**: No modifications to `doc/scapy/layers/tcp.rst`, `doc/scapy/usage.rst`, or any other existing documentation file.
- **Sphinx build configuration changes**: No changes to `doc/scapy/conf.py`, `.readthedocs.yml`, or `tox.ini` documentation environments.
- **New Packet class definitions**: Prohibited by the user's constraint.
- **`bind_layers()` calls**: Prohibited by the user's constraint.
- **Direct `StringBuffer` import or instantiation**: Prohibited by the user's constraint (the test uses `TCPSession` which internally creates `StringBuffer`; no direct user-level access).
- **Unrelated protocol documentation**: Only TCP reassembly overlap behavior is in scope. Other protocols (TLS, Kerberos, DCE/RPC, PostgreSQL) that implement `tcp_reassemble` are mentioned only as context, not investigated.

## 0.9 Execution Parameters

### 0.9.1 Documentation-Specific Instructions

- **Runtime environment**: Python 3.12.3 with Scapy installed from source (`pip install -e .` against commit `0925ada4`)
- **Test execution command**: `python3 /tmp/tcp_overlap_test.py` (the self-contained test script)
- **Default format**: Markdown with Mermaid diagrams and fenced code blocks
- **Citation requirement**: Every technical claim must reference a specific source file and line number (e.g., `scapy/sessions.py:193`)
- **Style guide**: The document follows the implementation rule structure — a comprehensive Q&A with thinking/rationale, grounded in the codebase
- **Documentation validation**: Manual review of terminal output against source code analysis; no automated linting or link-checking required for this standalone document

### 0.9.2 Key Runtime Observations to Document

The following runtime observations were collected during the investigation and must be included verbatim in the output document:

**Variant A (Identical-byte overlap)**:

- Seg1 payload (seq 201, pos 0–19): `b'HTTP/1.0 200 OK\r\nCon'`
- Seg2 payload (seq 211, pos 10–29): `b'00 OK\r\nContent-Lengt'`
- Seg3 payload (seq 231, pos 30–41): `b'h: 4\r\n\r\nABCD'`
- Overlap region [10:20) — From seg1: `b'00 OK\r\nCon'` / From seg2: `b'00 OK\r\nCon'` (identical)
- Reassembled bytes: `b'HTTP/1.0 200 OK\r\nContent-Length: 4\r\n\r\nABCD'`
- Reassembled length: 42 bytes
- Winner: seg2 (last writer), net effect: no visible change

**Variant B (Different-byte overlap)**:

- Seg1 payload (seq 201, pos 0–19): `b'HTTP/1.0 200 OK\r\nCon'`
- Seg2 payload (seq 211, pos 10–29): `b'ZZZZZZZZZZtent-Lengt'`
- Seg3 payload (seq 231, pos 30–41): `b'h: 4\r\n\r\nABCD'`
- Overlap region [10:20) — From seg1: `b'00 OK\r\nCon'` / From seg2: `b'ZZZZZZZZZZ'` (different)
- Reassembled bytes (packet 1): `b'HTTP/1.0 2ZZZZZZZZZZtent-Lengt'` (30 bytes)
- Reassembled bytes (packet 2): `b'h: 4\r\n\r\nABCD'` (12 bytes)
- Winner: seg2 (last writer), net effect: HTTP header corruption splits the stream

**Policy identification**: Last-writer-wins via unconditional `memoryview` assignment at `scapy/sessions.py:193`:
```python
memoryview(self.content)[seq:seq + data_len] = data
```

**Cleanup verification**: All three temporary files (`/tmp/_overlap_identical.pcap`, `/tmp/_overlap_different.pcap`, `/tmp/tcp_overlap_test.py`) confirmed deleted with `exists? False`.

## 0.10 Rules for Documentation

The following documentation-specific rules are derived from the user's explicit instructions and the implementation rule:

- **Do not modify any existing files in the source repository** — The output is a new file only (`blitzy/documentation/scapy_0925ada48540.md`). No existing source, test, or documentation file may be changed.
- **Base all answers on the code as truth** — Every claim in the document must be grounded in the actual Scapy source code at commit `0925ada4`. No theoretical explanations or assumptions about "typical" TCP reassembly behavior from external references.
- **Provide thinking / rationale behind the answers** — The document must explain *why* specific experimental choices were made (e.g., why HTTP on port 80, why three segments with 10-byte overlap, why the FIN flag triggers completion).
- **Use only built-in Scapy layers** — The test script must not define new `Packet` classes, must not call `bind_layers()`, and must not import or instantiate `StringBuffer` directly (even for analysis purposes).
- **Include verbatim terminal excerpts** — Runtime evidence must be exact, copy-paste-ready terminal output, not paraphrased or reformatted. The excerpts must include both the command executed and the output produced.
- **Clean up all temporary artifacts** — Temporary scripts and pcap files created during the investigation must be deleted after the run, with terminal evidence showing they are gone.
- **Place the output document in `blitzy/documentation/`** — The document filename must match the source branch name: `scapy_0925ada48540.md`.
- **Include source code citations** — Every reference to the Scapy codebase must include the file path and line number (e.g., `scapy/sessions.py:193`).
- **Two overlap variants required** — The document must cover both identical-byte overlap (retransmission with same data) and different-byte overlap (conflicting data for the same sequence range), with separate runtime evidence for each.

## 0.11 References

### 0.11.1 Codebase Files and Folders Searched

The following files and folders were examined across the repository to derive all conclusions in this Agent Action Plan:

| Path | Type | Purpose of Examination |
|------|------|----------------------|
| `` (root) | Folder | Repository structure discovery, top-level files |
| `scapy/` | Folder | Core package structure, module inventory |
| `scapy/sessions.py` | File | **Primary**: `StringBuffer` (line 161), `TCPSession` (line 223), overlap policy at line 193 |
| `scapy/layers/http.py` | File | `HTTP.tcp_reassemble()` (line 580), `guess_payload_class()` (line 637), `bind_layers` (line 750) |
| `scapy/sendrecv.py` | File | `sniff()` (line 1308), `AsyncSniffer._run()` (line 981), offline pcap path (lines 1107–1143) |
| `scapy/utils.py` | File | `wrpcap()` (line 1095), `PcapReader` (line 1367) — pcap I/O |
| `scapy/layers/inet.py` | File | `TCP` class (line 753), `IP` class — packet construction |
| `scapy/layers/` | Folder | Layer module inventory, `tcp_reassemble` implementations search |
| `scapy/contrib/postgres.py` | File | Additional `tcp_reassemble` implementation (contextual) |
| `scapy/layers/dcerpc.py` | File | Additional `tcp_reassemble` implementation (contextual) |
| `scapy/layers/kerberos.py` | File | Additional `tcp_reassemble` implementation (contextual) |
| `scapy/layers/tls/session.py` | File | Additional `tcp_reassemble` implementation (contextual) |
| `scapy/config.py` | File | `conf.load_layers` — identifies that HTTP is NOT in default layer list |
| `scapy/all.py` | File | Umbrella import surface |
| `scapy/layers/all.py` | File | Layer loading mechanism via `conf.load_layers` |
| `doc/` | Folder | Documentation infrastructure discovery |
| `doc/scapy/` | Folder | Sphinx source tree structure |
| `doc/scapy/conf.py` | File | Sphinx build configuration (lines 1–30) |
| `doc/scapy/layers/tcp.rst` | File | Existing TCP documentation — 67 lines, no TCPSession coverage |
| `doc/scapy/usage.rst` | File | TCPSession usage docs (lines 780–830) |
| `doc/scapy/index.rst` | File | Master toctree — navigation structure |
| `pyproject.toml` | File | Python version requirements, optional dependencies (docs extras), build configuration |
| `tox.ini` | File | Testing environments, docs build command, Python version matrix |
| `.readthedocs.yml` | File | ReadTheDocs hosting configuration |
| `README.md` | File | Project overview |
| `CONTRIBUTING.md` | File | Contributor guidelines |

### 0.11.2 Tech Spec Sections Consulted

| Section | Purpose |
|---------|---------|
| 1.1 Executive Summary | Project overview, version, license, stakeholders |
| 1.3 Scope | In-scope/out-of-scope boundaries, platform coverage, protocol domains |
| 3.3 Frameworks & Libraries | Zero-dependency architecture, optional dependencies including docs extras |
| 4.4 Session Management and Reassembly | TCP reassembly flow diagrams, StringBuffer description, metadata dictionary |
| 5.2 Component Details | Session Management component description, Protocol Stack architecture |
| 8.6 Documentation Infrastructure | ReadTheDocs config, Sphinx build chain, API doc generation |

### 0.11.3 Attachments and External Resources

- **Attachments provided**: None (0 attachments)
- **Figma URLs**: None
- **External URLs**: None referenced or required — all analysis is code-driven from the repository at commit `0925ada4` on branch `scapy_0925ada48540`

