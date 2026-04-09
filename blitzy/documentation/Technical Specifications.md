# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

### 0.1.1 Core Documentation Objective

Based on the provided requirements, the Blitzy platform understands that the documentation objective is to **create new documentation** that comprehensively answers a set of investigative questions about how Scapy's automotive ISO-TP (ISO 15765-2) module handles message segmentation over CAN frames. The user is onboarding to the Scapy codebase and requires a reference document that explains ISO-TP fragmentation behavior through both code analysis and runtime execution evidence.

**Documentation Category:** Create new documentation
**Documentation Type:** Technical Q&A / Investigative Reference Guide

The specific documentation requirements are:

- **R1 — Environment Setup:** Document the procedure to set up Scapy and load the ISO-TP module from the automotive contrib packages (`scapy.contrib.isotp`).
- **R2 — Segmentation for >1 CAN Frame Payloads:** Explain what happens when a diagnostic payload exceeds the capacity of a single CAN frame (7 data bytes for normal addressing).
- **R3 — 20-Byte Payload Frame Enumeration:** Show the actual CAN frames generated when fragmenting a 20-byte payload, including their count and hex content, backed by runtime execution evidence.
- **R4 — First Bytes of Each Frame:** Document the Protocol Control Information (PCI) bytes at the start of each generated CAN frame (First Frame header, Consecutive Frame sequence numbers).
- **R5 — Single-Frame Threshold:** Identify the maximum payload size before ISO-TP segmentation is triggered.
- **R6 — Absurdly Large Payload Behavior (5000 bytes):** Document the behavior and frame-generation outcome when attempting to send a 5000-byte payload, including whether the 32-bit FF_DL escape sequence is used.
- **R7 — Missing Frame Timeout and Error:** Identify the timeout value that applies when a middle Consecutive Frame never arrives during reassembly, and the error/warning message that results.
- **R8 — No Repository Modifications:** The investigation must not modify any repository files; only temporary scripts are permitted (and must be cleaned up).

### 0.1.2 Special Instructions and Constraints

- **CRITICAL — Read-Only Repository Constraint:** The user explicitly stated: User Example: "Don't modify any repository files; temporary scripts are fine but clean them up afterward." This means the output document is the sole deliverable; no source files in the Scapy repository may be altered.
- **Runtime Evidence Requirement:** The user specifically requested "verify this by providing runtime execution evidence," meaning the document must include actual script output proving the stated behaviors rather than purely theoretical analysis.
- **Implementation Rule — AtlasQnA-Repo:** The governing rule mandates: create a new markdown document named `<source_branch_name>.md` that comprehensively answers the question(s) posed, providing thinking/rationale behind the answers, basing all answers on the code as truth, and placing the document in the `blitzy/documentation` directory in the destination repo. The source branch name is `scapy_0925ada48540`.
- **Temporary Script Cleanup:** Any scripts created for runtime evidence must be deleted after execution.

### 0.1.3 Technical Interpretation

These documentation requirements translate to the following technical documentation strategy:

- To document ISO-TP setup (R1), we will **create** a section in `blitzy/documentation/scapy_0925ada48540.md` explaining the `load_contrib('isotp')` entry point and the `conf.contribs['ISOTP']` configuration, referencing `scapy/contrib/isotp/__init__.py`.
- To document segmentation behavior (R2), we will **analyze** the `ISOTP.fragment()` method in `scapy/contrib/isotp/isotp_packet.py` (lines 92–152) and explain the First Frame / Consecutive Frame splitting logic.
- To enumerate frames for 20 bytes (R3), we will **execute** a temporary Python script that creates `ISOTP(rx_id=0x641, data=bytes(range(0x01,0x15)))`, calls `.fragment()`, and captures the output. This runtime evidence will be embedded in the document.
- To explain PCI byte structure (R4), we will **reference** the ISO-TP frame type constants (`N_PCI_SF=0x00`, `N_PCI_FF=0x10`, `N_PCI_CF=0x20`, `N_PCI_FC=0x30`) from `isotp_packet.py` lines 42–45 and correlate them with the runtime output.
- To identify the segmentation threshold (R5), we will **analyze** the condition at `isotp_packet.py` line 106: `if len(self.data) <= data_bytes_in_frame` where `data_bytes_in_frame = 7` (or 6 with extended addressing).
- To document 5000-byte behavior (R6), we will **execute** a fragment call on a 5000-byte payload and document the 32-bit FF_DL escape sequence (`isotp_packet.py` lines 120–123).
- To document timeout behavior (R7), we will **reference** `isotp_soft_socket.py` lines 496–497 (`self.cf_timeout = 1`) and the `_rx_timer_handler` method at lines 619–629 that resets state and logs `"RX state was reset due to timeout"`.

### 0.1.4 Inferred Documentation Needs

Based on code analysis, the following implicit documentation needs have been identified:

- **Extended Addressing Impact:** The user did not ask about extended addressing, but the fragment() method has a distinct code path where `data_bytes_in_frame` changes from 7 to 6 when `rx_ext_address` is set. This affects segmentation thresholds and should be mentioned for completeness.
- **ISO-TP 2015 32-Bit FF_DL Escape:** The 5000-byte question naturally touches on the ISO 15765-2:2016 extension where payloads between 4096 and 4,294,967,295 bytes use a 32-bit length escape in the First Frame header. This mechanism is distinct from the original 12-bit FF_DL and should be documented.
- **Defragmentation Behavior with Missing Frames:** In addition to the socket-level timeout, the `ISOTPMessageBuilder` in `isotp_utils.py` returns `None` when defragmenting an incomplete set of CAN frames. Both paths should be documented.
- **UDS Relationship:** Since UDS (`scapy/contrib/automotive/uds.py`) subclasses `ISOTP`, the document should briefly note that diagnostic UDS payloads inherit the same fragmentation behavior.


## 0.2 Documentation Discovery and Analysis

### 0.2.1 Existing Documentation Infrastructure Assessment

Repository analysis reveals a mature Sphinx-based documentation ecosystem with comprehensive automotive protocol coverage, though no standalone ISO-TP segmentation deep-dive document exists.

**Documentation Framework:**

| Component | Detail | Source |
|---|---|---|
| Generator | Sphinx `>=3.0.0` | `pyproject.toml` `[project.optional-dependencies]` docs group |
| Theme | `sphinx_rtd_theme>=0.4.3` | `pyproject.toml` docs extra |
| Hosting | ReadTheDocs (`scapy.readthedocs.io`) | `.readthedocs.yml` |
| Build OS | Ubuntu 20.04, Python 3.9 | `.readthedocs.yml` |
| Source Root | `doc/scapy/` | Sphinx `conf.py` at `doc/scapy/conf.py` |
| Master TOC | `doc/scapy/index.rst` | Sphinx landing page with toctree |
| Build Entry | `doc/scapy/Makefile`, `doc/scapy/make.bat` | Platform-specific build wrappers |
| Extensions | `autodoc`, `napoleon`, `todo`, `linkcode`, plus custom `_ext/` | `doc/scapy/conf.py` lines 37–41 |
| API Docs | `sphinx-apidoc` via `apitree` tox environment | `tox.ini` |
| Diagram Tools | Mermaid (not in Sphinx config; inline RST figures with static PNG/SVG) | `doc/scapy/graphics/automotive/` |

**Documentation Generator Configuration:** `doc/scapy/conf.py` is the central Sphinx configuration module. It imports version metadata from the `scapy` package and enables extensions including `autodoc` and `napoleon` for docstring extraction.

### 0.2.2 Repository Code Analysis for Documentation

Search patterns used for identifying code to document:

- **ISO-TP Packet Definitions:** `scapy/contrib/isotp/isotp_packet.py` — Contains the `ISOTP` class, `fragment()`, `defragment()`, all four frame types (`ISOTP_SF`, `ISOTP_FF`, `ISOTP_CF`, `ISOTP_FC`), and protocol constants.
- **ISO-TP Soft Socket (State Machine):** `scapy/contrib/isotp/isotp_soft_socket.py` — Contains `ISOTPSoftSocket`, `ISOTPSocketImplementation` with timeout values (`fc_timeout=1`, `cf_timeout=1`), the `_rx_timer_handler`, and `_tx_timer_handler` methods.
- **ISO-TP Utils (Reassembly):** `scapy/contrib/isotp/isotp_utils.py` — Contains `ISOTPMessageBuilder`, `ISOTPSession`, and `Bucket`-based reassembly logic.
- **ISO-TP Package Init (Backend Selection):** `scapy/contrib/isotp/__init__.py` — Contains backend selection logic (`ISOTPNativeSocket` vs. `ISOTPSoftSocket`), the `USE_CAN_ISOTP_KERNEL_MODULE` flag, and all re-exports.
- **CAN Layer:** `scapy/layers/can.py` — Defines the base `CAN` packet class, `CAN_MAX_DLEN=8`, and CAN frame structure.
- **UDS Layer:** `scapy/contrib/automotive/uds.py` — UDS subclasses `ISOTP` directly, inheriting segmentation behavior.
- **ISO-TP Scanner:** `scapy/contrib/isotp/isotp_scanner.py` — Active ISO-TP discovery on CAN buses.
- **ISO-TP Native Socket:** `scapy/contrib/isotp/isotp_native_socket.py` — Linux kernel CAN_ISOTP backend via ctypes.

Key directories examined:

| Directory | Relevance |
|---|---|
| `scapy/contrib/isotp/` | Primary: all ISO-TP protocol logic, sockets, utils |
| `scapy/contrib/automotive/` | Context: UDS, DoIP, KWP that use ISO-TP as transport |
| `scapy/layers/` | Foundation: CAN packet class |
| `doc/scapy/layers/` | Existing: `automotive.rst` covers ISO-TP at overview level |
| `doc/scapy/` | Context: Sphinx config, manual structure |

**Related existing documentation found:**

- `doc/scapy/layers/automotive.rst` — The main automotive documentation page. It covers ISO-TP at a conceptual level (frame types, protocol flow, socket types, scan utilities) but does **not** include a detailed walkthrough of actual frame generation for specific payloads or runtime evidence of segmentation behavior. This gap is precisely what the user seeks.
- `doc/notebooks/` — Contains tutorial notebooks (Scapy in 15 minutes, HTTP/2, TLS) but no automotive/ISO-TP-specific notebook.
- `README.md` — Project overview; no ISO-TP detail.

### 0.2.3 Web Search Research Conducted

No external web search was required for this documentation task. All answers are derivable from the codebase itself (`isotp_packet.py`, `isotp_soft_socket.py`, `isotp_utils.py`), validated through runtime execution of the installed Scapy package. The ISO 15765-2 protocol mechanics are encoded directly in the source code constants and logic.


## 0.3 Documentation Scope Analysis

### 0.3.1 Code-to-Documentation Mapping

**Module: `scapy/contrib/isotp/isotp_packet.py`**

- Public APIs requiring documentation:
  - `ISOTP` class — Packet class for ISO-TP messages with slots for `tx_id`, `rx_id`, `ext_address`, `rx_ext_address`
  - `ISOTP.fragment()` — Fragments an ISOTP message into CAN frames (lines 92–152)
  - `ISOTP.defragment()` — Static method to reassemble CAN frames into an ISOTP message (lines 154–195)
  - `ISOTPHeader` — CAN-based header with PCI nibble dispatch (lines 198–238)
  - `ISOTPHeaderEA` — Extended addressing variant (lines 241–262)
  - `ISOTP_SF`, `ISOTP_FF`, `ISOTP_CF`, `ISOTP_FC` — The four ISO-TP frame type classes (lines 271–309)
- Constants: `CAN_MAX_DLEN=8`, `ISOTP_MAX_DLEN=4095`, `ISOTP_MAX_DLEN_2015=4294967295`, `N_PCI_SF/FF/CF/FC`
- Current documentation: Partially covered in `doc/scapy/layers/automotive.rst` (protocol overview, basic usage). **Missing:** detailed fragmentation walkthrough, frame byte analysis, threshold documentation.
- Documentation needed: Detailed Q&A on segmentation mechanics, runtime evidence of frame generation.

**Module: `scapy/contrib/isotp/isotp_soft_socket.py`**

- Key elements for documentation:
  - `ISOTPSocketImplementation.__init__()` — Sets `fc_timeout=1` and `cf_timeout=1` (line 496–497)
  - `_rx_timer_handler()` — Resets `rx_state` to `ISOTP_IDLE` on consecutive frame timeout, logs warning (lines 619–629)
  - `_tx_timer_handler()` — Resets `tx_state` to `ISOTP_IDLE` on flow control timeout, logs warning (lines 631–643)
  - `_recv_ff()` — Schedules `cf_timeout` timer after receiving First Frame (line 842–843)
  - `_recv_cf()` — Re-schedules `cf_timeout` timer for each expected Consecutive Frame (lines 910–911)
- Current documentation: `automotive.rst` mentions ISOTPSoftSocket usage but **not** timeout values or error messages.
- Documentation needed: Timeout values, error/warning message text, state machine reset behavior.

**Module: `scapy/contrib/isotp/isotp_utils.py`**

- Key elements for documentation:
  - `ISOTPMessageBuilder` — Reassembly class using `Bucket` objects (lines 59–321)
  - `ISOTPMessageBuilder.Bucket` — Tracks `total_len`, `current_len`, `ready` state (lines 83–105)
  - `ISOTP.defragment()` — Returns `None` when incomplete frames provided
- Current documentation: Not directly documented in user-facing docs.
- Documentation needed: Behavior when frames are missing (returns None).

**Module: `scapy/contrib/isotp/__init__.py`**

- Key elements: Backend selection logic, `load_contrib('isotp')` entry point, `USE_CAN_ISOTP_KERNEL_MODULE` flag
- Current documentation: Covered in `automotive.rst` (ISOTPSocket configuration)
- Documentation needed: Setup procedure as part of the Q&A document

### 0.3.2 Documentation Gap Analysis

Given the requirements and repository analysis, documentation gaps include:

| Gap Area | Current State | Required Coverage |
|---|---|---|
| Segmentation threshold (7 bytes) | Not explicitly documented | Explain threshold with code reference and evidence |
| Frame-by-frame analysis for specific payloads | Not documented | Show exact CAN frames for 20-byte payload with hex dump |
| PCI byte structure per frame type | Explained in RST overview diagrams only | Show actual first-byte values from runtime output |
| 32-bit FF_DL escape (ISO 15765-2:2016) | Not documented | Demonstrate with 5000-byte payload |
| Upper size limits and exceptions | Not documented | Document `ISOTP_MAX_DLEN_2015` and `Scapy_Exception` |
| Consecutive Frame timeout value | Not documented | Document `cf_timeout=1` second and warning message |
| Defragmentation with missing frames | Not documented | Show `None` return from `ISOTP.defragment()` |
| Extended addressing impact on thresholds | Partially in automotive.rst | Note threshold drops from 7 to 6 bytes |


## 0.4 Documentation Implementation Design

### 0.4.1 Documentation Structure Planning

The deliverable is a single comprehensive Markdown document placed per the implementation rule:

    blitzy/
    └── documentation/
        └── scapy_0925ada48540.md

**Internal structure of `scapy_0925ada48540.md`:**

#### ISO-TP Message Segmentation in Scapy's Automotive Diagnostic Protocols

#### Environment Setup
#### ISO-TP Segmentation Fundamentals

#### Maximum Payload Before Segmentation
#### Fragmenting a 20-Byte Diagnostic Payload

#### Handling Large Payloads (5000 Bytes)
#### Absolute Size Limits

#### Receiving Segmented Messages — Missing Frame Behavior
#### Summary Table

#### Source References

### 0.4.2 Content Generation Strategy

**Information Extraction Approach:**

- Extract fragmentation logic from `scapy/contrib/isotp/isotp_packet.py` lines 92–152 (the `fragment()` method) for threshold analysis and frame construction rules.
- Extract timeout constants from `scapy/contrib/isotp/isotp_soft_socket.py` lines 496–497 (`fc_timeout=1`, `cf_timeout=1`) and error handler text from lines 619–629 and 631–643.
- Generate runtime evidence by executing temporary Python scripts against the installed Scapy package, capturing actual fragment output for 1–9 byte payloads, 20-byte payloads, and 5000-byte payloads.
- Create diagram material by mapping the ISO-TP First Frame → Flow Control → Consecutive Frame exchange using the state constants from `isotp_soft_socket.py` (lines 47–51: `ISOTP_IDLE`, `ISOTP_WAIT_FIRST_FC`, `ISOTP_WAIT_FC`, `ISOTP_WAIT_DATA`, `ISOTP_SENDING`).

**Documentation Standards:**

- Markdown formatting with proper headers (`#`, `##`, `###`)
- Code examples using fenced Python code blocks with syntax highlighting
- Source citations as inline references: `Source: scapy/contrib/isotp/isotp_packet.py:106`
- Tables for parameter descriptions and frame breakdowns
- Mermaid diagrams for the ISO-TP segmentation flow

### 0.4.3 Diagram and Visual Strategy

**Mermaid diagrams to create:**

- **Segmentation Decision Flowchart:** Flowchart showing the decision path in `fragment()` — payload size check → Single Frame vs. First Frame + Consecutive Frames, including the 12-bit vs. 32-bit FF_DL branch.
- **ISO-TP Frame Exchange Sequence:** Sequence diagram showing Sender→Receiver flow for a multi-frame transmission: First Frame → Flow Control → Consecutive Frames, with timeout annotation.
- **PCI Byte Structure Table:** Visual representation of the first byte layout for SF (0x0L), FF (0x1H 0xLL), CF (0x2N), and FC (0x3F).

All diagrams will be authored using Mermaid syntax within fenced code blocks for direct rendering in Markdown viewers.


## 0.5 Documentation File Transformation Mapping

### 0.5.1 File-by-File Documentation Plan

| Target Documentation File | Transformation | Source Code/Docs | Content/Changes |
|---|---|---|---|
| `blitzy/documentation/scapy_0925ada48540.md` | CREATE | `scapy/contrib/isotp/isotp_packet.py`, `scapy/contrib/isotp/isotp_soft_socket.py`, `scapy/contrib/isotp/isotp_utils.py`, `scapy/contrib/isotp/__init__.py`, `scapy/layers/can.py`, `scapy/contrib/automotive/uds.py` | Complete investigative Q&A document covering ISO-TP setup, segmentation thresholds, 20-byte payload frame analysis, 5000-byte payload behavior, size limits, timeout values, missing-frame behavior — all with runtime execution evidence |

**Transformation Mode Summary:**
- **CREATE** — 1 file: `blitzy/documentation/scapy_0925ada48540.md`
- **UPDATE** — 0 files (no existing files are modified per user constraint)
- **DELETE** — 0 files
- **REFERENCE** — 1 file: `doc/scapy/layers/automotive.rst` (used as style/context reference for automotive documentation)

### 0.5.2 New Documentation File Detail

**File: `blitzy/documentation/scapy_0925ada48540.md`**

- **Type:** Technical Q&A / Investigative Reference
- **Source Code:**
  - `scapy/contrib/isotp/isotp_packet.py` — `ISOTP.fragment()`, `ISOTP.defragment()`, frame type classes, constants
  - `scapy/contrib/isotp/isotp_soft_socket.py` — `ISOTPSocketImplementation`, timeout values, state machine handlers
  - `scapy/contrib/isotp/isotp_utils.py` — `ISOTPMessageBuilder`, `Bucket`, reassembly logic
  - `scapy/contrib/isotp/__init__.py` — Backend selection, `load_contrib` entry point
  - `scapy/layers/can.py` — `CAN` packet class, `CAN_MAX_DLEN=8`
  - `scapy/contrib/automotive/uds.py` — UDS subclassing ISOTP
- **Sections:**
  - Environment Setup (`load_contrib('isotp')`, `conf.contribs['ISOTP']`)
  - Segmentation Fundamentals (four frame types, CAN data limit)
  - Maximum Payload Before Segmentation (7 bytes normal, 6 bytes extended, runtime evidence)
  - 20-Byte Payload Fragmentation (3 CAN frames, hex dump, PCI analysis, runtime evidence)
  - 5000-Byte Payload Behavior (715 CAN frames, 32-bit FF_DL escape, runtime evidence)
  - Absolute Size Limits (`ISOTP_MAX_DLEN_2015`, `Scapy_Exception`)
  - Missing Frame Timeout (`cf_timeout=1s`, warning message, `None` return from defragment)
  - Summary Table (quick-reference answers to all questions)
  - Source References (file paths and line numbers)
- **Diagrams:**
  - Mermaid flowchart: segmentation decision logic
  - Mermaid sequence diagram: multi-frame ISO-TP exchange
- **Key Citations:**
  - `scapy/contrib/isotp/isotp_packet.py:92-152` (fragment method)
  - `scapy/contrib/isotp/isotp_packet.py:103-104` (size limit check)
  - `scapy/contrib/isotp/isotp_packet.py:106` (single-frame threshold)
  - `scapy/contrib/isotp/isotp_soft_socket.py:496-497` (timeout values)
  - `scapy/contrib/isotp/isotp_soft_socket.py:619-629` (RX timeout handler)

### 0.5.3 Documentation Configuration Updates

No documentation configuration files need updating. This task creates a standalone Markdown document in `blitzy/documentation/` which is outside the Sphinx documentation tree. No changes to `doc/scapy/index.rst`, `mkdocs.yml`, or `.readthedocs.yml` are required.

### 0.5.4 Cross-Documentation Dependencies

- **Shared Context:** The new document references the same ISO-TP protocol concepts described in `doc/scapy/layers/automotive.rst` but does not duplicate that content. It provides a deeper, evidence-based investigation complementary to the existing overview.
- **No Navigation Links Required:** The output file is placed in `blitzy/documentation/` which is a separate documentation namespace from the Scapy Sphinx tree.
- **No Index/Glossary Updates:** No updates to existing indices or glossaries are needed.


## 0.6 Dependency Inventory

### 0.6.1 Documentation Dependencies

| Registry | Package Name | Version | Purpose |
|---|---|---|---|
| pip | scapy | 2026.04.09 (repo HEAD) | Core library under investigation; provides ISO-TP, CAN, and automotive contrib modules |
| pip | setuptools | >=62.0.0 | Build backend required by `pyproject.toml` for editable install |
| stdlib | struct | (Python 3.12 stdlib) | Used internally by `isotp_packet.py` for PCI byte packing |
| stdlib | logging | (Python 3.12 stdlib) | Used internally by `isotp_soft_socket.py` for timeout warning messages |

**Runtime Environment:**

| Component | Version | Source |
|---|---|---|
| Python | 3.12.3 | System runtime (compatible with `requires-python = ">=3.7, <4"` from `pyproject.toml`) |
| Scapy | 2026.04.09 | Installed via `pip install -e .` from repository root |
| OS | Linux (container) | Required for `scapy.consts.LINUX` checks in `isotp/__init__.py` |

**Note on CAN Hardware Dependencies:**

The ISO-TP fragmentation logic (`ISOTP.fragment()`) operates purely in-memory on packet objects and does not require CAN hardware, kernel modules, or the `python-can` library. The runtime evidence was gathered by calling `fragment()` directly on `ISOTP` objects without any socket or bus interaction. The `python-can` package and Linux `can-isotp` kernel module are only required for live CAN bus communication, which is out of scope for this documentation task.

### 0.6.2 Documentation Reference Updates

No documentation link updates are required. The new document `blitzy/documentation/scapy_0925ada48540.md` is a self-contained artifact that references source files by their repository-relative paths. No existing documentation files contain links that need to be updated.


## 0.7 Coverage and Quality Targets

### 0.7.1 Documentation Coverage Metrics

**Current coverage analysis (before this task):**

| Documentation Area | Documented | Total | Coverage |
|---|---|---|---|
| ISO-TP frame types explained | 4/4 | 4 | 100% (in `automotive.rst`) |
| Segmentation threshold documented | 0/2 | 2 | 0% (normal + extended addressing) |
| Runtime fragmentation evidence | 0/3 | 3 | 0% (20-byte, 5000-byte, boundary cases) |
| Timeout values documented | 0/2 | 2 | 0% (cf_timeout, fc_timeout) |
| Missing-frame behavior documented | 0/2 | 2 | 0% (socket timeout, defragment return) |
| Size limit documentation | 0/2 | 2 | 0% (12-bit, 32-bit FF_DL) |
| **Overall user questions answered** | **0/7** | **7** | **0%** |

**Target coverage (after this task): 100%** — All 7 user questions fully answered with runtime evidence.

**Coverage gaps to address:**

| Gap | Source Module | Target Coverage |
|---|---|---|
| Single-frame payload threshold | `isotp_packet.py:106` | Document both 7-byte (normal) and 6-byte (extended) limits |
| Frame-by-frame hex output for 20 bytes | `isotp_packet.py:fragment()` | Full hex dump with PCI byte annotation |
| 5000-byte payload frame count and FF_DL | `isotp_packet.py:120-123` | 715 frames, 32-bit escape sequence |
| Consecutive frame timeout value | `isotp_soft_socket.py:497` | `cf_timeout = 1` second |
| Timeout warning message text | `isotp_soft_socket.py:629` | `"RX state was reset due to timeout"` |
| Defragment with missing frames | `isotp_utils.py` / `isotp_packet.py:154-195` | Returns `None` |
| ISO-TP setup procedure | `isotp/__init__.py` | `load_contrib('isotp')` workflow |

### 0.7.2 Documentation Quality Criteria

**Completeness Requirements:**
- Every user question must have a direct, explicit answer in the document
- Each answer must include a source code citation (file path and line number)
- Runtime evidence (actual script output) must be provided for all fragmentation-related questions
- The segmentation threshold boundary must be demonstrated with 7-byte and 8-byte payloads

**Accuracy Validation:**
- All runtime output was generated by executing Python scripts against the repository's installed Scapy package (version 2026.04.09)
- Frame hex values were captured from actual `ISOTP.fragment()` calls, not manually constructed
- Timeout values were read directly from `isotp_soft_socket.py` source code
- Warning message text was copied verbatim from the `_rx_timer_handler` method

**Clarity Standards:**
- Each section begins with a direct answer before providing technical detail
- Hex dumps include per-byte annotation (PCI type, length, sequence number, data)
- A summary table provides quick-reference answers to all user questions
- Terminology is consistent with ISO 15765-2 standard names used in the Scapy source

**Maintainability:**
- Source citations include file paths and line numbers for traceability
- Runtime evidence is reproducible by running the documented script snippets
- The document structure follows a question-answer pattern that maps directly to the user's original questions

### 0.7.3 Example and Diagram Requirements

| Requirement | Target |
|---|---|
| Code examples | Minimum 3: setup, 20-byte fragmentation, 5000-byte fragmentation |
| Runtime output captures | 3: threshold boundary, 20-byte frames, 5000-byte First Frame |
| Mermaid diagrams | 2: segmentation decision flowchart, multi-frame exchange sequence |
| Tables | 3: frame hex breakdown, summary answers, PCI byte reference |
| Code example testing | All examples verified via actual Python execution during analysis |


## 0.8 Scope Boundaries

### 0.8.1 Exhaustively In Scope

**New documentation files:**
- `blitzy/documentation/scapy_0925ada48540.md` — The sole deliverable, a comprehensive Markdown Q&A document

**Source code files analyzed (read-only):**
- `scapy/contrib/isotp/isotp_packet.py` — Fragment/defragment logic, frame types, constants
- `scapy/contrib/isotp/isotp_soft_socket.py` — Timeout values, state machine, error handlers
- `scapy/contrib/isotp/isotp_utils.py` — ISOTPMessageBuilder reassembly logic
- `scapy/contrib/isotp/__init__.py` — Backend selection, load_contrib entry point
- `scapy/contrib/isotp/isotp_scanner.py` — Referenced for completeness
- `scapy/contrib/isotp/isotp_native_socket.py` — Referenced for completeness
- `scapy/layers/can.py` — CAN packet class and CAN_MAX_DLEN constant
- `scapy/contrib/automotive/uds.py` — UDS subclass relationship to ISOTP
- `scapy/contrib/automotive/__init__.py` — Automotive namespace init

**Existing documentation analyzed (read-only, for context):**
- `doc/scapy/layers/automotive.rst` — Existing automotive documentation
- `doc/scapy/conf.py` — Sphinx configuration
- `doc/scapy/index.rst` — Documentation structure
- `.readthedocs.yml` — Documentation hosting configuration
- `pyproject.toml` — Package metadata and dependency declarations
- `tox.ini` — Test and CI environment configuration
- `README.md` — Project overview

**Temporary artifacts (created and cleaned up):**
- `/tmp/isotp_investigation.py` — Runtime evidence script (executed and deleted)

**Topics exhaustively documented:**
- ISO-TP module loading and configuration
- Single-frame vs. multi-frame segmentation threshold (7 bytes / 6 bytes)
- 20-byte payload fragmentation with full frame hex dump and PCI analysis
- 5000-byte payload fragmentation with 32-bit FF_DL escape sequence
- Absolute size limits (`ISOTP_MAX_DLEN=4095`, `ISOTP_MAX_DLEN_2015=4294967295`)
- Consecutive frame timeout value (`cf_timeout=1` second)
- Missing frame error behavior (warning message, state reset, `None` return)
- Extended addressing impact on segmentation thresholds

### 0.8.2 Explicitly Out of Scope

- **Source code modifications** — No changes to any file in the Scapy repository (per explicit user instruction)
- **Test file modifications** — No changes to `test/` directory files
- **Feature additions or code refactoring** — No code changes of any kind
- **Deployment configuration changes** — No changes to CI/CD, `.readthedocs.yml`, or `tox.ini`
- **Existing documentation updates** — No changes to `doc/scapy/layers/automotive.rst` or any other existing docs
- **Sphinx/RST documentation generation** — The output is standalone Markdown, not part of the Sphinx build
- **Live CAN bus testing** — No hardware interaction; all evidence is from in-memory fragment() calls
- **ISO-TP scanner functionality** — The `isotp_scan` tool is not part of the user's questions
- **DoIP, GMLAN, SOME/IP, CCP, XCP protocols** — Other automotive protocols are out of scope
- **Python-can library integration** — Not needed for fragmentation-only analysis
- **ISOTPNativeSocket (kernel module) details** — Kernel-level behavior is not part of the user's questions
- **CAN FD (Flexible Data-Rate) considerations** — The user's questions reference standard CAN frames


## 0.9 Execution Parameters

### 0.9.1 Documentation-Specific Instructions

| Parameter | Value |
|---|---|
| **Output format** | Markdown (`.md`) with Mermaid diagrams in fenced code blocks |
| **Output location** | `blitzy/documentation/scapy_0925ada48540.md` |
| **Citation requirement** | Every technical claim must reference source file path and line number |
| **Style guide** | Question-answer format matching the user's investigative sequence |
| **Runtime evidence method** | Execute temporary Python scripts via `python3`, capture stdout, embed in document |
| **Script cleanup** | All temporary scripts must be deleted after execution (verified: `/tmp/isotp_investigation.py` was removed) |
| **Repository modification** | Strictly prohibited — read-only access to all repository files |
| **Diagram format** | Mermaid syntax within fenced code blocks for native Markdown rendering |
| **Code example format** | Python snippets with syntax highlighting, maximum 5-10 lines per example |

### 0.9.2 Runtime Evidence Collection Protocol

The following protocol was used to collect runtime evidence:

- **Step 1:** Install Scapy from the repository source using `pip install -e .` from the repo root at `/tmp/blitzy/scapy/scapy_0925ada48540_ca08b7/`
- **Step 2:** Write a temporary investigation script at `/tmp/isotp_investigation.py`
- **Step 3:** Execute the script with `python3 /tmp/isotp_investigation.py` and capture all stdout output
- **Step 4:** Verify the output contains:
  - Threshold boundary test (1–9 byte payloads → single/multi-frame transition at 8 bytes)
  - Full frame hex dump for 20-byte payload (3 frames: FF + CF1 + CF2)
  - 5000-byte payload frame count (715 frames) and 32-bit FF_DL header
  - Defragmentation with missing frames returning `None`
- **Step 5:** Delete the temporary script: `rm /tmp/isotp_investigation.py` (verified)
- **Step 6:** Embed the captured output in the documentation file as runtime evidence blocks

### 0.9.3 Documentation Validation

- **Link checking:** Not applicable (standalone Markdown, no cross-links)
- **Code example validation:** All Python snippets verified by actual execution
- **Factual accuracy:** All values cross-referenced against source code (line numbers documented)
- **Completeness check:** Every user question has a dedicated section with direct answer, rationale, and evidence


## 0.10 Rules for Documentation

### 0.10.1 User-Specified Rules

The following rules were explicitly specified by the user and must be strictly observed:

- **"Don't modify any repository files; temporary scripts are fine but clean them up afterward."**
  - No file in the Scapy repository (under `scapy/`, `doc/`, `test/`, `.config/`, `.github/`, or root-level files) may be created, modified, or deleted.
  - Temporary scripts may be created in `/tmp/` for runtime evidence collection but must be deleted after use.
  - The sole output file is `blitzy/documentation/scapy_0925ada48540.md` in the destination repo.

- **"Verify this by providing runtime execution evidence."**
  - The document must contain actual output from executing Scapy code, not theoretical analysis alone.
  - Runtime evidence must demonstrate the fragmentation of a 20-byte payload and show the actual generated CAN frames.

### 0.10.2 Implementation Rule — SWE-AtlasQnA-Repo

The governing implementation rule specifies:

- **Create a new markdown document** named `<source_branch_name>.md` — resolved to `scapy_0925ada48540.md`
- **Comprehensively answer** the question(s) posed in the prompt
- **Provide thinking/rationale** behind the answers
- **Do not make assumptions** — base all answers on the code as the truth
- **Do not modify any existing files** in the source repository
- **Place the generated document** in the `blitzy/documentation` directory in the destination repo

### 0.10.3 Documentation Quality Rules

- All hex values must be presented in consistent format (e.g., `0x10`, `0x14`)
- PCI byte analysis must align with ISO 15765-2 terminology (Single Frame, First Frame, Consecutive Frame, Flow Control)
- Source citations must use the format: `Source: <filepath>:<line_number>`
- Runtime output must be presented in code blocks with clear labels indicating they are execution output, not code
- Each answer must begin with a concise direct statement before elaborating with technical detail
- Consistent use of Scapy class names as they appear in code (`ISOTP`, `ISOTP_SF`, `ISOTP_FF`, `ISOTP_CF`, `ISOTP_FC`, `ISOTPSoftSocket`, etc.)


## 0.11 References

### 0.11.1 Repository Files Searched and Analyzed

The following files and folders were systematically searched and analyzed to derive all conclusions in this Agent Action Plan:

**ISO-TP Core Module (Primary Sources):**

| File | Lines Analyzed | Key Information Extracted |
|---|---|---|
| `scapy/contrib/isotp/isotp_packet.py` | 1–309 (complete) | `ISOTP` class, `fragment()` method (lines 92–152), `defragment()` method (lines 154–195), frame type classes (`ISOTP_SF`, `ISOTP_FF`, `ISOTP_CF`, `ISOTP_FC`), constants (`ISOTP_MAX_DLEN=4095`, `ISOTP_MAX_DLEN_2015=4294967295`, `CAN_MAX_DLEN=8`, `N_PCI_SF/FF/CF/FC`) |
| `scapy/contrib/isotp/isotp_soft_socket.py` | 1–980 (complete) | `ISOTPSoftSocket` class, `ISOTPSocketImplementation` with `fc_timeout=1` and `cf_timeout=1` (lines 496–497), `_rx_timer_handler` (lines 619–629), `_tx_timer_handler` (lines 631–643), `_recv_ff` (lines 792–843), `_recv_cf` (lines 845–911), `TimeoutScheduler` class |
| `scapy/contrib/isotp/isotp_utils.py` | 1–349 (complete) | `ISOTPMessageBuilder` class, `Bucket` helper class, `ISOTPSession`, reassembly logic, `_feed_first_frame`, `_feed_consecutive_frame`, sequence number tracking |
| `scapy/contrib/isotp/__init__.py` | 1–48 (complete) | Backend selection (`ISOTPNativeSocket` vs `ISOTPSoftSocket`), `USE_CAN_ISOTP_KERNEL_MODULE` flag, `load_contrib` entry point, re-exports |
| `scapy/contrib/isotp/isotp_scanner.py` | (summary only) | Active ISO-TP endpoint discovery — out of scope for this task |
| `scapy/contrib/isotp/isotp_native_socket.py` | (summary only) | Linux kernel CAN_ISOTP backend — referenced for completeness |

**Supporting Modules:**

| File | Lines Analyzed | Key Information Extracted |
|---|---|---|
| `scapy/layers/can.py` | 1–80 | `CAN` packet class, `CAN_MAX_DLEN=8`, `CAN_MAX_IDENTIFIER`, frame structure |
| `scapy/contrib/automotive/uds.py` | 1–60 | UDS subclasses ISOTP directly (`class UDS(ISOTP)`), service ID enumeration |
| `scapy/contrib/automotive/__init__.py` | (summary only) | Automotive namespace marker, logger setup |

**Documentation Files:**

| File | Key Information Extracted |
|---|---|
| `doc/scapy/layers/automotive.rst` | Lines 1–950: ISO-TP protocol overview, frame types, socket types, usage examples, CAN background, diagnostic protocol stack |
| `doc/scapy/conf.py` | Sphinx configuration, extensions, build settings |
| `doc/scapy/index.rst` | Documentation structure and toctree |
| `doc/scapy/layers/index.rst` | Layer documentation index with glob pattern |

**Configuration and Build Files:**

| File | Key Information Extracted |
|---|---|
| `pyproject.toml` | Python `>=3.7, <4`, optional deps (sphinx, sphinx_rtd_theme, tox), build backend |
| `.readthedocs.yml` | Python 3.9, Ubuntu 20.04, EPUB/PDF outputs |
| `tox.ini` | `py38-isotp_kernel_module` test environment, CAN/ISOTP integration testing |
| `setup.py` | Legacy build script with VERSION generation |
| `README.md` | Project overview, installation, resource links |

**Folder Structure Explored:**

| Folder | Depth | Purpose |
|---|---|---|
| `` (root) | 1 | Repository root — identified all top-level files and folders |
| `scapy/` | 2 | Core package — identified all subpackages and modules |
| `scapy/contrib/` | 3 | Contrib extensions — identified isotp/ and automotive/ |
| `scapy/contrib/isotp/` | 4 | ISO-TP package — read all 6 files |
| `scapy/contrib/automotive/` | 3 | Automotive package — identified UDS and related modules |
| `doc/` | 2 | Documentation root — identified Sphinx tree |
| `doc/scapy/` | 3 | Sphinx source — identified all RST chapters and assets |
| `doc/scapy/layers/` | 4 | Layer docs — identified automotive.rst and all layer pages |

### 0.11.2 Attachments

No attachments were provided by the user. No Figma screens or external design assets are associated with this task.

### 0.11.3 Runtime Evidence Artifacts

| Artifact | Location | Status |
|---|---|---|
| `/tmp/isotp_investigation.py` | Temporary script | Created, executed, and **deleted** per user constraint |
| Runtime output (stdout) | Captured in analysis context | Embedded in Agent Action Plan for reference; will be included in final document |

**Key runtime findings captured:**

- 7-byte payload → 1 CAN frame (Single Frame); 8-byte payload → 2 CAN frames (First + Consecutive)
- 20-byte payload → 3 CAN frames: FF (`1014010203040506`), CF1 (`210708090a0b0c0d`), CF2 (`220e0f1011121314`)
- 5000-byte payload → 715 CAN frames; First Frame uses 32-bit escape: `100000001388bbbb`
- Defragmentation with missing frames returns `None`
- Consecutive Frame timeout: `cf_timeout = 1` second; warning: `"RX state was reset due to timeout"`


