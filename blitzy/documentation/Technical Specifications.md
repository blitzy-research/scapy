# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

### 0.1.1 Core Documentation Objective

Based on the provided requirements, the Blitzy platform understands that the documentation objective is to **create new documentation** that comprehensively answers a series of investigative questions about how the Scapy packet-manipulation library represents and encodes IPv4 protocol fields at runtime. The request is categorized as:

- **Category:** Create new documentation
- **Documentation Type:** Technical investigation report / Q&A document
- **Target Artifact:** A single Markdown file named `scapy_0925ada48540.md`, placed in the `blitzy/documentation` directory of the destination repository

The user's requirements decompose into four distinct investigative tasks:

- **Requirement 1 — Serialize a valid IPv4 packet:** Construct an IPv4 packet with a destination address, a TTL value, and at least one flag; serialize it in a running environment; report the total byte length and the first few bytes in hexadecimal as actually observed at runtime
- **Requirement 2 — Inspect the destination field's Python type:** Access the `dst` field on the constructed packet and state the Python type it exposes during execution
- **Requirement 3 — Test an invalid IPv4 destination:** Repeat the experiment with a clearly invalid IPv4 destination and report what happens in practice — at what point an error appears and in what form
- **Requirement 4 — Trace the field class in the codebase:** Identify the field class responsible for handling IPv4 addresses, including the method that converts Python values into on-the-wire bytes, the validation it performs, and how this class participates in Scapy's broader field type system or registry

**Implicit Documentation Needs:**

- The document must include rationale and thinking behind each answer, not just raw results
- All answers must be grounded in actual code execution and codebase inspection — no assumptions
- The error call chain for invalid input must be traced through the actual source files
- The field type hierarchy (from `Field` base class through `IPField` and its subclasses) must be explained to contextualize IPv4 address handling within Scapy's broader field system
- Mermaid diagrams should illustrate the field conversion pipeline and class hierarchy

### 0.1.2 Special Instructions and Constraints

- **CRITICAL — Read-only constraint:** "Do not modify any source files." The generated document must be the only new artifact. No changes to any file under the `scapy/` source tree, `doc/`, `test/`, or any other existing directory
- **CRITICAL — Implementation rule:** Per the `SWE-AtlasQnA-Repo` rule, the output document must be named `<source_branch_name>.md` — concretely `scapy_0925ada48540.md` — and placed in the `blitzy/documentation` directory
- **Temporary scripts only:** "Only use temporary scripts if required, and ensure any test data is cleaned up." Any scripts written for runtime experiments must be placed under `/tmp/` and removed after use
- **No existing file modifications:** "Do not modify any existing files in the source repository." This is explicitly restated in the implementation rule
- **Evidence-based answers:** "Do not make assumptions, base your answers on the code as the truth." Every claim must cite specific source file paths and line numbers
- **Include rationale:** "Provide thinking / rationale behind the answers." The document must explain the *why*, not just the *what*

### 0.1.3 Technical Interpretation

These documentation requirements translate to the following technical documentation strategy:

- To answer the valid-packet serialization question, we will **create a section** in `blitzy/documentation/scapy_0925ada48540.md` documenting the runtime output of constructing `IP(dst='192.168.1.1', ttl=128, flags='DF')`, calling `raw()`, and reporting the 20-byte result with hexadecimal representation
- To document the Python type of the destination field, we will **create a section** explaining that `pkt.dst` returns a Python `str` (from `builtins` module), traced through `IPField.i2h()` in `scapy/fields.py`
- To document invalid-input behavior, we will **create a section** detailing the `socket.gaierror` exception raised during packet construction (not serialization), tracing the call chain: `Packet.__init__()` → `IPField.any2i()` → `IPField.h2i()` → `inet_aton()` (fails) → `Net()` constructor → `socket.getaddrinfo()` → `gaierror`
- To document the field class system, we will **create a section** explaining `IPField` (at `scapy/fields.py:796`), its `i2m()` method for wire conversion, `h2i()` for validation, and its position in the `Field` class hierarchy with 43+ direct subclasses

### 0.1.4 Inferred Documentation Needs

- Based on code analysis: The `IPField` class at `scapy/fields.py:796` is the primary field handler, but `DestIPField` (at `scapy/layers/inet.py:503`) and `SourceIPField` (at `scapy/fields.py:854`) are specialized subclasses used in the actual `IP` packet definition that require documentation
- Based on structure: The conversion pipeline (`h2i` → `i2m` → `addfield` for building; `getfield` → `m2i` → `i2h` for dissection) spans `scapy/fields.py` and `scapy/packet.py`, requiring a consolidated explanation
- Based on dependencies: The `Net` class at `scapy/base_classes.py:112` is a critical dependency of `IPField.h2i()` that handles CIDR notation and hostname resolution, and must be documented as part of the validation chain
- Based on the field type system: The `Field_metaclass` at `scapy/base_classes.py:404` and the generic `Field` base class at `scapy/fields.py:139` form the registry backbone that all 43+ field types inherit from

## 0.2 Documentation Discovery and Analysis

### 0.2.1 Existing Documentation Infrastructure Assessment

Repository analysis reveals a **mature Sphinx-based documentation system** with comprehensive coverage of general usage but no dedicated IPv4 field-internals investigation document.

- **Documentation framework:** Sphinx `>=3.0.0` with `sphinx_rtd_theme>=0.4.3`
- **Documentation generator configuration:** `doc/scapy/conf.py` — configures extensions (`autodoc`, `napoleon`, `todo`, `linkcode`, custom `scapy_doc`), import paths, and output targets (HTML, LaTeX, man, Texinfo)
- **Documentation source root:** `doc/scapy/` — contains 15 `.rst` files and 4 subdirectories
- **API documentation tools:** `sphinx-apidoc` via the `apitree` tox environment (see `tox.ini`)
- **Diagram tools:** Mermaid (not currently used in the Sphinx docs; the docs use static SVGs in `doc/scapy/graphics/`)
- **Hosting/deployment:** ReadTheDocs via `.readthedocs.yml` — builds on Ubuntu 20.04 with Python 3.9, outputs HTML + EPUB + PDF
- **Build command:** `sphinx-build -W --keep-going -b html . _build/html` (CI docs job)

**Existing documentation files discovered:**

| File | Purpose | Relevance to This Task |
|------|---------|----------------------|
| `doc/scapy/build_dissect.rst` | Explains adding new protocols, field conversion (`i2m`, `m2i`, `h2i`, `addfield`, `getfield`), and field type listing | HIGH — contains the authoritative explanation of the field state model (i/m/h) |
| `doc/scapy/usage.rst` | Interactive tutorial: packet crafting, sending, sniffing, analysis | MEDIUM — covers packet construction but not field internals |
| `doc/scapy/introduction.rst` | Philosophy, quick demo, sensible defaults | LOW — general background |
| `doc/scapy/installation.rst` | Platform install, optional deps, pyreverse diagram generation hint | LOW — notes `pyreverse -o png -p fields scapy/fields.py` |
| `doc/scapy/routing.rst` | Interface enumeration, IPv4/IPv6 routes | LOW — routing not field encoding |
| `README.md` | Project overview, quick-start, links to docs | LOW — no field internals |
| `CONTRIBUTING.md` | Contributor guide, testing, protocol placement | LOW — process not implementation |

**No existing document covers the specific questions posed** (runtime byte output, Python type of `dst`, invalid-input error behavior, `IPField` class internals, or field type registry participation). The output document will be entirely new.

### 0.2.2 Repository Code Analysis for Documentation

Search patterns used to identify code relevant to the documentation questions:

- **Field definitions:** `scapy/fields.py` — 3868 lines, contains 60+ field classes including `IPField` (line 796), `SourceIPField` (line 854), `DestField` (line 729)
- **IPv4 layer definition:** `scapy/layers/inet.py` — contains `IP` class (line 521) with `fields_desc`, `DestIPField` (line 503)
- **Base metaclass and Net:** `scapy/base_classes.py` — `Field_metaclass` (line 404), `Net` class (line 112), `Packet_metaclass` (line 281)
- **Core packet infrastructure:** `scapy/packet.py` — `Packet.__init__()` (line 141+), `build()`, `self_build()`, `post_build()`
- **IP address conversion utilities:** `scapy/utils.py` — `inet_aton` (line 625), `inet_ntoa` (line 634)
- **IPv6 portable fallbacks:** `scapy/pton_ntop.py` — `inet_pton`, `inet_ntop` with fallback implementations

Key directories examined:
- `scapy/` (root package — 34 modules)
- `scapy/layers/` (protocol layers — 48+ modules)
- `doc/scapy/` (Sphinx documentation source — 15 `.rst` files)
- `doc/notebooks/` (Jupyter tutorial notebooks)

Related documentation found:
- `doc/scapy/build_dissect.rst` lines 128–175: explains the `i/m/h` field state model and the `addfield`/`getfield` API
- `doc/scapy/build_dissect.rst` lines 1066–1069: lists `IPField` and `SourceIPField` as TCP/IP field types (bare listing, no explanation)

### 0.2.3 Web Search Research Conducted

No external web search was required for this documentation task. All questions are answerable directly from the codebase and runtime execution. The existing `build_dissect.rst` documentation provides the authoritative framing for Scapy's field conversion model, and the runtime experiments provide the empirical evidence needed for each answer.

## 0.3 Documentation Scope Analysis

### 0.3.1 Code-to-Documentation Mapping

**Modules requiring documentation coverage in the output document:**

- **Module: `scapy/fields.py`**
  - Public APIs to document: `Field` base class (line 139), `IPField` class (line 796), `SourceIPField` class (line 854), `DestField` class (line 729)
  - Key methods: `h2i()`, `i2m()`, `m2i()`, `i2h()`, `any2i()`, `addfield()`, `getfield()`, `randval()`
  - Current documentation: Partially covered in `doc/scapy/build_dissect.rst` (general field model) but `IPField` specifics undocumented
  - Documentation needed: Detailed walkthrough of `IPField.i2m()` wire conversion, `IPField.h2i()` validation pipeline, class hierarchy position

- **Module: `scapy/layers/inet.py`**
  - Public APIs to document: `IP` class (line 521) with full `fields_desc`, `DestIPField` class (line 503)
  - Key elements: `IP.fields_desc` list (14 fields), `IP.post_build()` (checksum/length computation)
  - Current documentation: Usage examples in `doc/scapy/usage.rst` but no internals exposition
  - Documentation needed: Field layout table, serialization walkthrough for a sample packet, `DestIPField` dual-inheritance explanation

- **Module: `scapy/base_classes.py`**
  - Public APIs to document: `Net` class (line 112), `Field_metaclass` class (line 404), `Packet_metaclass.__call__()` (line 379)
  - Current documentation: Not documented in existing docs
  - Documentation needed: Role of `Net` in DNS resolution fallback during `IPField.h2i()`, metaclass role in the field/packet registration system

- **Module: `scapy/packet.py`**
  - Public APIs to document: `Packet.__init__()` (field initialization loop at line 180+), `Packet.build()` / `self_build()` / `post_build()`
  - Current documentation: General build/dissect lifecycle in `build_dissect.rst`
  - Documentation needed: The specific `__init__` → `any2i` call path that triggers validation

- **Module: `scapy/utils.py`**
  - Public APIs to document: `inet_aton` wrapper (line 625), `inet_ntoa` alias (line 634)
  - Current documentation: Not explicitly documented
  - Documentation needed: The workaround for Python bug 643005 regarding `inet_aton("255.255.255.255")`

### 0.3.2 Documentation Gap Analysis

Given the requirements and repository analysis, documentation gaps relevant to the output document include:

- **No existing document answers the user's four questions** — all content must be created from scratch
- **Undocumented runtime behavior:** The precise byte output of `raw(IP(dst='192.168.1.1', ttl=128, flags='DF'))` and the Python type returned by `pkt.dst` are not documented anywhere
- **Undocumented error path:** The error chain for invalid IPv4 addresses (`Packet.__init__` → `any2i` → `h2i` → `inet_aton` → `Net()` → `socket.getaddrinfo` → `gaierror`) is not documented
- **Undocumented IPField internals:** While `build_dissect.rst` lists `IPField` as a field type, it does not explain its `i2m()` implementation, the `inet_aton`-based conversion, or the `Net` fallback in `h2i()`
- **Undocumented field type registry:** The `Field_metaclass`, the 43+ direct `Field` subclasses, and how `IPField` fits into this hierarchy are not covered in existing documentation
- **No existing blitzy/documentation directory:** The target directory does not exist and must be created

## 0.4 Documentation Implementation Design

### 0.4.1 Documentation Structure Planning

The output is a single Markdown file. Its planned internal structure:

```
blitzy/
└── documentation/
    └── scapy_0925ada48540.md
        ├── Introduction (context and approach)
        ├── Section 1: Valid IPv4 Packet Serialization
        │   ├── Packet construction and field values
        │   ├── Serialized bytes: length and hex dump
        │   ├── Byte-by-byte breakdown of the 20-byte header
        │   └── Rationale: how post_build() computes checksum/length
        ├── Section 2: Python Type of the Destination Field
        │   ├── Runtime observation: type(pkt.dst) == str
        │   ├── Rationale: IPField.i2h() returns str via cast
        │   └── Internal representation vs. human representation
        ├── Section 3: Invalid IPv4 Destination Behavior
        │   ├── Runtime observation: gaierror at construction time
        │   ├── Detailed error call chain with file:line citations
        │   ├── Why serialization is never reached
        │   └── Rationale: h2i() validation via inet_aton + Net fallback
        ├── Section 4: The IPField Class and Field Type System
        │   ├── IPField class location and signature
        │   ├── i2m(): the Python-to-wire conversion method
        │   ├── h2i(): validation via inet_aton and Net
        │   ├── Subclasses: SourceIPField, DestIPField
        │   ├── The Field base class and its conversion API
        │   ├── Field_metaclass and registration
        │   ├── Field type hierarchy diagram (Mermaid)
        │   └── IPField's role in the broader field system
        └── Conclusion
```

### 0.4.2 Content Generation Strategy

**Information Extraction Approach:**

- "Extract `IPField` method signatures and implementation from `scapy/fields.py:796-852` using direct file reading"
- "Extract the `IP` class `fields_desc` from `scapy/layers/inet.py:524-537` to show the complete IPv4 field layout"
- "Generate runtime examples by executing temporary Python scripts in the project's virtual environment"
- "Create class hierarchy diagrams by inspecting `Field.__subclasses__()` at runtime and mapping the MRO of `IPField` and `DestIPField`"
- "Trace the error call chain by examining the traceback output from `IP(dst='999.999.999.999')`"

**Documentation Standards:**

- Markdown formatting with `#`, `##`, `###` headers
- Mermaid diagrams for the field conversion pipeline and class hierarchy, enclosed in triple-backtick `mermaid` blocks
- Code examples using triple-backtick `python` blocks with syntax highlighting
- Source citations as inline references: `Source: scapy/fields.py:796`
- Tables for byte-level breakdowns and field-type comparisons
- Consistent use of Scapy terminology: "internal" (i), "machine" (m), "human" (h) representations

### 0.4.3 Diagram and Visual Strategy

**Mermaid diagrams to create within the output document:**

- **Field conversion pipeline flowchart:** Showing the `h2i → i2m → addfield` build path and `getfield → m2i → i2h` dissection path, specific to `IPField`
- **IPField class hierarchy diagram:** Showing `Field` → `IPField` → `SourceIPField` / `DestIPField`, with `DestField` as a mixin for `DestIPField`
- **Error call chain diagram:** Showing the sequence from `IP(dst='invalid')` through `__init__` → `any2i` → `h2i` → `inet_aton` (fail) → `Net()` → `getaddrinfo` → `gaierror`
- **IPv4 header byte layout diagram:** A table or visual showing the 20-byte IPv4 header structure as serialized by `raw()`

## 0.5 Documentation File Transformation Mapping

### 0.5.1 File-by-File Documentation Plan

| Target Documentation File | Transformation | Source Code/Docs | Content/Changes |
|---------------------------|----------------|------------------|-----------------|
| `blitzy/documentation/scapy_0925ada48540.md` | CREATE | `scapy/fields.py`, `scapy/layers/inet.py`, `scapy/base_classes.py`, `scapy/packet.py`, `scapy/utils.py`, `scapy/pton_ntop.py` | Complete Q&A investigation document covering IPv4 field representation, serialization output, Python types, invalid-input error behavior, and the IPField class role in the field type system |

**Transformation Modes Applied:**

- **CREATE** — `blitzy/documentation/scapy_0925ada48540.md`: The sole new artifact. This is the only file to be created in the entire task
- **REFERENCE** — `doc/scapy/build_dissect.rst`: Used as a stylistic and factual reference for explaining the field conversion model (i/m/h states). Not modified
- **REFERENCE** — `doc/scapy/usage.rst`: Used as context for how Scapy packet construction is typically documented. Not modified

No UPDATE or DELETE operations are required. No existing files are modified.

### 0.5.2 New Documentation File Detail

```
File: blitzy/documentation/scapy_0925ada48540.md
Type: Technical Investigation / Q&A Report
Source Code:
  - scapy/fields.py (lines 139-310: Field base; lines 729-756: DestField; lines 796-852: IPField; lines 854-890: SourceIPField)
  - scapy/layers/inet.py (lines 503-520: DestIPField; lines 521-600: IP class)
  - scapy/base_classes.py (lines 112-180: Net class; lines 281-403: Packet_metaclass; lines 404-415: Field_metaclass)
  - scapy/packet.py (lines 141-210: Packet.__init__)
  - scapy/utils.py (lines 620-634: inet_aton/inet_ntoa wrappers)
Sections:
  - Introduction (context, methodology, environment)
  - Section 1: Valid IPv4 Serialization (runtime output, byte-level breakdown, rationale)
  - Section 2: Destination Field Python Type (runtime type check, IPField.i2h explanation)
  - Section 3: Invalid IPv4 Behavior (runtime error, full call chain trace, rationale)
  - Section 4: IPField Class and Field Type System (class anatomy, i2m method, h2i validation, hierarchy, registry)
  - Conclusion (summary of findings)
Diagrams:
  - Mermaid class diagram: Field → IPField → SourceIPField / DestIPField hierarchy
  - Mermaid flowchart: field conversion pipeline (h2i → i2m → addfield)
  - Mermaid sequence diagram: error call chain for invalid IPv4 input
Key Citations:
  - scapy/fields.py:796 (IPField class definition)
  - scapy/fields.py:830-835 (IPField.i2m — wire conversion)
  - scapy/fields.py:803-810 (IPField.h2i — validation logic)
  - scapy/layers/inet.py:521-537 (IP class fields_desc)
  - scapy/layers/inet.py:503-520 (DestIPField class)
  - scapy/base_classes.py:112-180 (Net class — DNS fallback)
  - scapy/packet.py:186-190 (field initialization loop calling any2i)
  - scapy/utils.py:625-634 (inet_aton wrapper)
```

### 0.5.3 Documentation Configuration Updates

No documentation configuration files require modification. The output document is a standalone Markdown file in `blitzy/documentation/` and does not integrate with the existing Sphinx documentation system in `doc/scapy/`.

### 0.5.4 Cross-Documentation Dependencies

- **No shared content/includes:** The output document is self-contained
- **No navigation links:** The document does not link into the existing Sphinx toctree
- **No table of contents updates:** No modification to `doc/scapy/index.rst`
- **No index/glossary updates:** Not applicable

## 0.6 Dependency Inventory

### 0.6.1 Documentation Dependencies

The documentation task requires only the Scapy runtime to execute experiments. No additional documentation tooling (Sphinx, MkDocs, etc.) is needed since the output is a standalone Markdown file.

| Registry | Package Name | Version | Purpose |
|----------|--------------|---------|---------|
| pip | scapy | 2.7.0 (installed as editable from repo) | Runtime environment for executing IPv4 packet serialization experiments and inspecting field types |
| pip | setuptools | >=62.0.0 | Build backend required for editable install of Scapy (`pyproject.toml` build-system) |
| stdlib | socket | (Python 3.12 stdlib) | Provides `inet_aton`, `inet_ntoa`, `getaddrinfo` used by `IPField` for address validation and conversion |
| stdlib | struct | (Python 3.12 stdlib) | Binary packing/unpacking used by the `Field` base class `addfield`/`getfield` |

**No external documentation generation tools are required.** The output is hand-authored Markdown, not generated from source. The Sphinx infrastructure (`sphinx>=3.0.0`, `sphinx_rtd_theme>=0.4.3`) in `pyproject.toml` `[docs]` extra is irrelevant to this task.

### 0.6.2 Documentation Reference Updates

Not applicable. The output document is a new standalone file in `blitzy/documentation/` and does not modify or reference any existing documentation links. No link transformation rules are needed.

## 0.7 Coverage and Quality Targets

### 0.7.1 Documentation Coverage Metrics

**Current coverage analysis (before this task):**

- User-posed questions documented: 0/4 (0%) — none of the four investigative questions have answers in the existing documentation
- `IPField` internals documented: 0/6 key methods (0%) — `h2i`, `i2m`, `m2i`, `i2h`, `any2i`, `randval` are not explained in existing docs
- Field type system documented: Partial — `build_dissect.rst` describes the general `i/m/h` model but does not enumerate the full hierarchy or explain `Field_metaclass`

**Target coverage (after this task):**

- User-posed questions documented: 4/4 (100%)
- `IPField` methods explained in context: `h2i()`, `i2m()`, `m2i()`, `i2h()`, `any2i()` — all five conversion methods relevant to the investigation
- Field type hierarchy coverage: `Field` base → `IPField` → `SourceIPField`, `DestField` → `DestIPField` — complete chain documented with MRO

**Coverage gaps addressed:**

| Gap | Current State | Target State |
|-----|--------------|--------------|
| Runtime serialization byte output | Not documented anywhere | Fully documented with hex dump and byte breakdown |
| Python type of `pkt.dst` | Not documented | Documented with runtime evidence and source tracing |
| Invalid-input error behavior | Not documented | Full call chain traced with file:line citations |
| `IPField.i2m()` wire conversion | Listed in `build_dissect.rst` field catalog only | Fully explained with code walkthrough |
| `IPField.h2i()` validation | Not documented | Explained including `inet_aton` → `Net` fallback |
| Field type hierarchy | Not documented | Class diagram + narrative explanation |

### 0.7.2 Documentation Quality Criteria

**Completeness requirements:**

- Every answer includes the runtime output as actually observed (not theoretical)
- Every claim cites a specific source file path and line number
- The error call chain includes every intermediate function in the stack trace
- The field class explanation covers the full inheritance chain from `Field` to `DestIPField`

**Accuracy validation:**

- Code examples are verified by actual execution in the project's virtual environment (Python 3.12, Scapy editable install)
- Hex byte output matches actual `raw()` output captured during runtime experiments
- Error messages and exception types match actual traceback output
- Line numbers match the specific commit (`0925ada4`)

**Clarity standards:**

- Technical accuracy with accessible language — explain Scapy's `i/m/h` field model before using the terminology
- Progressive disclosure — start with simple runtime observations, then trace into source code internals
- Consistent terminology: use "internal representation" (i), "machine representation" (m), "human representation" (h) throughout

**Maintainability:**

- Source citations use `file:line` format for traceability
- All runtime experiments are reproducible with the documented commands

### 0.7.3 Example and Diagram Requirements

- **Minimum examples:** Each of the 4 questions receives at least one complete, runnable code example with output
- **Diagram types required:** Mermaid class diagram (field hierarchy), Mermaid flowchart (conversion pipeline), Mermaid sequence diagram (error chain)
- **Code example testing:** All examples verified by execution in the virtual environment at `/tmp/scapy_env/`
- **Visual content:** All diagrams rendered as Mermaid blocks within the Markdown document

## 0.8 Scope Boundaries

### 0.8.1 Exhaustively In Scope

**New documentation files:**

- `blitzy/documentation/scapy_0925ada48540.md` — the sole deliverable

**Source files to analyze (read-only) for content extraction:**

- `scapy/fields.py` — `Field` base class, `IPField`, `SourceIPField`, `DestField`, `FlagsField`, `ByteField`, and the full field type catalog
- `scapy/layers/inet.py` — `IP` packet class, `DestIPField`, `fields_desc` layout, `post_build()` for checksum/length
- `scapy/base_classes.py` — `Net` class (DNS resolution / CIDR handling), `Field_metaclass`, `Packet_metaclass.__call__()`
- `scapy/packet.py` — `Packet.__init__()` field initialization loop, `build()` / `self_build()` / `do_build()` / `post_build()` lifecycle
- `scapy/utils.py` — `inet_aton` wrapper (bug 643005 workaround), `inet_ntoa` alias
- `scapy/pton_ntop.py` — Portable `inet_pton` / `inet_ntop` implementations (context for IPv6 comparison)

**Documentation assets to create:**

- Mermaid diagram: IPField class hierarchy
- Mermaid diagram: field conversion pipeline
- Mermaid diagram: invalid-input error sequence

**Runtime experiments to execute (in `/tmp/` only):**

- Construct `IP(dst='192.168.1.1', ttl=128, flags='DF')`, serialize, report bytes
- Inspect `type(pkt.dst)` on the constructed packet
- Attempt `IP(dst='999.999.999.999')` and capture the error
- Inspect `IPField.__mro__`, `Field.__subclasses__()` for hierarchy data

### 0.8.2 Explicitly Out of Scope

- **Source code modifications:** No file under `scapy/`, `doc/`, `test/`, or any other existing directory shall be modified, added to, or deleted
- **Test file modifications:** No test files are touched
- **Feature additions or code refactoring:** Not applicable — this is a documentation-only task
- **Deployment configuration changes:** No changes to `.readthedocs.yml`, `pyproject.toml`, `tox.ini`, or CI files
- **Sphinx documentation integration:** The output file is standalone Markdown; no `.rst` files are modified, no `index.rst` toctree is updated, no Sphinx build is affected
- **IPv6 field documentation:** While `IP6Field` is mentioned for context, a full investigation of IPv6 field handling is out of scope
- **Non-IPv4 protocol fields:** Fields for TCP, UDP, Ethernet, etc. are out of scope except as brief comparisons
- **Performance analysis:** Runtime performance of serialization/deserialization is not analyzed
- **Network I/O:** No packets are actually sent to the network; only `raw()` serialization is performed
- **Persistent test data:** Any temporary scripts are cleaned up; no persistent test artifacts remain

## 0.9 Execution Parameters

### 0.9.1 Documentation-Specific Instructions

- **Documentation build command:** Not applicable — the output is standalone Markdown, not Sphinx-generated. For reference, the existing Sphinx docs build via `cd doc/scapy && make html`
- **Documentation preview command:** Any Markdown renderer (e.g., `grip`, GitHub preview, VS Code Markdown Preview) can render the output file
- **Diagram generation command:** Mermaid diagrams are embedded inline in the Markdown and rendered by any Mermaid-capable viewer (GitHub natively renders Mermaid in `.md` files)
- **Default format:** Markdown with Mermaid diagrams
- **Citation requirement:** Every technical claim must reference a source file path and line number from the `scapy_0925ada48540` commit
- **Style guide:** Follow the `SWE-AtlasQnA-Repo` implementation rule — provide thinking/rationale, base answers on code, do not modify existing files
- **Runtime environment for experiments:**
  - Python 3.12.3 (system), virtual environment at `/tmp/scapy_env/`
  - Scapy installed as editable from `/tmp/blitzy/scapy/scapy_0925ada48540_ca08b7/`
  - No network access required (only `raw()` serialization, no packet sending)
- **Cleanup requirement:** All temporary scripts used for experiments must be created under `/tmp/` and removed after capturing output

## 0.10 Rules for Documentation

The following rules are derived from the user's explicit instructions and the `SWE-AtlasQnA-Repo` implementation rule:

- **"Do not modify any source files."** — No file in the repository's existing tree (`scapy/`, `doc/`, `test/`, `setup.py`, `pyproject.toml`, etc.) may be modified. The only new artifact is `blitzy/documentation/scapy_0925ada48540.md`
- **"Do not modify any existing files in the source repository."** — Restated from the implementation rule for emphasis. This applies to every file discovered via repository inspection
- **"Only use temporary scripts if required, and ensure any test data is cleaned up."** — Any Python scripts created for runtime experiments must reside in `/tmp/` and be deleted after use. No residual files may remain
- **"Do not make assumptions, base your answers on the code as the truth."** — Every claim about Scapy's behavior must be supported by either runtime output or a specific source file citation. Inferences from documentation alone are insufficient; the code is the authoritative source
- **"Provide thinking / rationale behind the answers."** — The output document must not merely state facts but explain why each behavior occurs, tracing from user-visible effects to underlying implementation details
- **"Create a new markdown document named `<source_branch_name>.md`."** — The file must be named exactly `scapy_0925ada48540.md` (matching the branch name `scapy_0925ada48540`)
- **"Place the generated document in the `blitzy/documentation` directory in the destination repo."** — The directory must be created if it does not exist. The full path is `blitzy/documentation/scapy_0925ada48540.md`

## 0.11 References

### 0.11.1 Codebase Files and Folders Searched

The following files and folders were inspected to derive the conclusions in this Agent Action Plan:

| Path | Type | Purpose in Analysis |
|------|------|-------------------|
| `` (root) | Folder | Top-level repository structure discovery |
| `pyproject.toml` | File | Python version constraints (`>=3.7, <4`), build system (`setuptools>=62.0.0`), optional dependencies (`docs` extra) |
| `.readthedocs.yml` | File | Documentation hosting configuration (Sphinx, Python 3.9, Ubuntu 20.04) |
| `tox.ini` | File | Test matrix and CI environments (Python versions 2.7–3.11) |
| `scapy/` | Folder | Core package structure — 34 modules, 7 subdirectories |
| `scapy/fields.py` | File | **Primary source**: `Field` base class (line 139), `IPField` (line 796), `SourceIPField` (line 854), `DestField` (line 729), `FlagsField` (line 2997); 3868 lines total, 60+ field types |
| `scapy/layers/inet.py` | File | **Primary source**: `IP` class (line 521), `DestIPField` (line 503), IPv4 `fields_desc` layout, `post_build()` checksum/length logic |
| `scapy/base_classes.py` | File | **Primary source**: `Net` class (line 112) for CIDR/hostname handling, `Field_metaclass` (line 404), `Packet_metaclass.__call__()` (line 379) |
| `scapy/packet.py` | File | `Packet.__init__()` field initialization loop (line 180+), `build()`/`self_build()` lifecycle |
| `scapy/utils.py` | File | `inet_aton` wrapper (line 625) with bug 643005 workaround, `inet_ntoa` alias (line 634) |
| `scapy/pton_ntop.py` | File | Portable `inet_pton`/`inet_ntop` fallback implementations for IPv4/IPv6 |
| `scapy/layers/` | Folder | Protocol layer catalog — 48+ modules; `inet.py` identified as IPv4 target |
| `doc/scapy/` | Folder | Sphinx documentation source tree — 15 `.rst` files, `conf.py`, build files |
| `doc/scapy/build_dissect.rst` | File | Existing documentation of field conversion model (`i/m/h` states), `addfield`/`getfield` API, field type listing (IPField at line 1068) |
| `doc/scapy/index.rst` | File | Sphinx toctree structure — confirmed `build_dissect` is in "Extend scapy" section |
| `doc/scapy/conf.py` | File | Sphinx configuration — extensions (`autodoc`, `napoleon`, `todo`, `linkcode`, `scapy_doc`), version metadata |
| `doc/scapy/usage.rst` | File | Usage tutorial — packet crafting examples (contextual reference) |
| `doc/scapy/installation.rst` | File | Installation docs — notes `pyreverse` for field diagram generation |
| `doc/scapy/layers/` | Folder | Protocol-specific documentation (automotive, Bluetooth, HTTP, etc.) — no IPv4-specific doc exists |
| `doc/notebooks/` | Folder | Jupyter tutorials — contextual reference only |
| `README.md` | File | Project overview, quick-start, documentation links |
| `CONTRIBUTING.md` | File | Contributor guidelines |

### 0.11.2 Runtime Experiments Conducted

| Experiment | Command | Key Result |
|-----------|---------|------------|
| Valid IPv4 serialization | `raw(IP(dst='192.168.1.1', ttl=128, flags='DF'))` | 20 bytes; hex `450000140001400080008640a9fe0901c0a80101` |
| Destination field type | `type(IP(dst='192.168.1.1').dst)` | `str` (Python built-in) |
| Invalid IP: `999.999.999.999` | `IP(dst='999.999.999.999')` | `socket.gaierror` at construction time (not serialization) |
| Invalid IP: `256.1.1.1` | `IP(dst='256.1.1.1')` | `socket.gaierror` at construction time |
| Invalid IP: non-resolvable hostname | `IP(dst='not_an_ip')` | `socket.gaierror` at construction time |
| Invalid IP: empty string | `IP(dst='')` | `socket.gaierror` at construction time |
| None destination | `IP(dst=None)` | Falls back to default `'127.0.0.1'` (via `DestIPField.dst_from_pkt`) |
| IPField MRO | `IPField.__mro__` | `IPField → Field → Generic → object` |
| DestIPField MRO | `DestIPField.__mro__` | `DestIPField → IPField → DestField → Field → Generic → object` |
| Field subclass count | `len(Field.__subclasses__())` | 43 direct subclasses |
| `inet_aton` validation | `socket.inet_aton('999.999.999.999')` | `OSError: illegal IP address string passed to inet_aton` |

### 0.11.3 Attachments

No attachments were provided by the user. No Figma URLs or external design files are referenced.

### 0.11.4 Tech Spec Sections Consulted

- **Section 1.1 — Executive Summary:** Scapy project overview, version 2.7.0, Python >=3.7 requirement
- **Section 3.2 — Programming Languages:** Python as sole language, version lifecycle, standard library modules used
- **Section 3.3 — Frameworks & Libraries:** Zero external dependencies, optional dependency groups, Sphinx docs extras
- **Section 5.2 — Component Details:** Core Packet Engine (packet.py, fields.py), Field API (`addfield`/`getfield`), build/dissect lifecycle sequence diagrams
- **Section 8.6 — Documentation Infrastructure:** ReadTheDocs configuration, Sphinx build configuration, API documentation generation via `sphinx-apidoc`

