# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification


### 0.1.1 Core Documentation Objective

Based on the provided requirements, the Blitzy platform understands that the documentation objective is to **create a new investigative documentation file** that comprehensively answers a series of interrelated questions about how Scapy's Ethernet layer (`Ether`) handles packet construction, padding, and unknown EtherType values. The document will be produced by building test packets in memory, observing Scapy's behavior, and tracing the relevant source code logic — all without modifying any repository files.

**Documentation Category:** Create new documentation

**Documentation Type:** Technical investigation / Q&A reference document

**Requirements with Enhanced Clarity:**

- **R1 — Normal Ethernet/IP/TCP construction**: Build a standard `Ether()/IP()/TCP()` packet and document its structure, field defaults, and total byte length when serialized via `raw()`.
- **R2 — Short-payload padding behavior**: Construct an Ethernet frame with a very small payload (approximately 10 bytes) and determine whether Scapy automatically pads the frame to the IEEE 802.3 minimum of 60 bytes (excluding FCS). Document whether padding occurs during the `build()` phase or only during dissection.
- **R3 — Raw roundtrip fidelity**: Serialize a short-payload packet to bytes via `raw()`, then re-parse it with `Ether(raw(packet))`, and document what happens to any padding — whether it persists, disappears, or transforms into a `Padding` layer.
- **R4 — Unknown EtherType behavior**: Construct an Ethernet frame using an EtherType that Scapy does not recognize (e.g., `0x9000`, `0xBEEF`, `0x1234`) and document what Scapy renders after the Ethernet layer during `show()` — specifically whether it dumps raw data, raises an error, or attempts protocol guessing.
- **R5 — Padding threshold discovery**: Test multiple payload sizes to identify the exact byte count at which Scapy transitions from adding padding (during dissection) to not needing it, referencing the 60-byte minimum Ethernet frame standard.
- **R6 — Source code tracing**: Identify and document the specific source code locations in Scapy where (a) padding decisions are made, (b) EtherType-to-protocol dispatch occurs, and (c) unknown protocols are handled. Provide rationale grounded in the actual code.

**Inferred Documentation Needs:**

- Based on code analysis: The `Ether` class in `scapy/layers/l2.py` (line 244) does **not** override `extract_padding()`, meaning the Ethernet layer itself never separates padding from payload — this responsibility falls to inner protocol layers such as `IP`, which uses its `len` field to compute padding boundaries. This critical distinction must be documented.
- Based on structure: The `Padding` class (`scapy/packet.py`, line 1906), the `conf.padding` setting (`scapy/config.py`, line 787), and the `build_padding()` method (`scapy/packet.py`, line 742) form a coordinated system for padding behavior that spans multiple files and must be consolidated in the documentation.
- Based on binding mechanism: The `bind_layers()` / `guess_payload_class()` / `default_payload_class()` chain (`scapy/packet.py`, lines 1062–1090 and 1974–1996) governs EtherType dispatch and needs clear documentation showing the fallback path to `conf.raw_layer` (which is `Raw`).
- Based on user journey: The document must include reproducible Python code examples, explain the behavioral "why" behind each observation, and trace each answer to specific source file lines.

### 0.1.2 Special Instructions and Constraints

- **CRITICAL: Read-only constraint** — The user explicitly states: "Don't modify any of the repository files while you're investigating." This means all investigation must be purely observational and the only file created is the output markdown document.
- **Implementation rule SWE-AtlasQnA-Repo** mandates:
  - The output document must be named `<source_branch_name>.md` → **`scapy_0925ada48540.md`**
  - It must be placed in the `blitzy/documentation/` directory in the destination repo
  - It must provide thinking/rationale behind the answers
  - All answers must be based on the code as truth — no assumptions
  - No existing files in the source repository may be modified
- **Style preference**: The document should include working code examples demonstrating each behavior, with clear explanations tied to specific source code references.

### 0.1.3 Technical Interpretation

These documentation requirements translate to the following technical documentation strategy:

- To **answer the padding questions (R1, R2, R3, R5)**, we will create comprehensive documentation sections in `blitzy/documentation/scapy_0925ada48540.md` that demonstrate Scapy's build-time vs. dissection-time behavior using reproducible code examples and trace the logic through `Packet.build()` → `build_padding()` → `do_build()` in `scapy/packet.py` and `IP.extract_padding()` in `scapy/layers/inet.py`.
- To **answer the unknown EtherType question (R4)**, we will document the `guess_payload_class()` → `payload_guess` list → `default_payload_class()` → `conf.raw_layer` (Raw) fallback chain in `scapy/packet.py`, cross-referenced with the `bind_layers(Ether, ...)` registrations scattered across `scapy/layers/*.py`.
- To **satisfy the source code tracing requirement (R6)**, we will cite exact file paths and line numbers for each mechanism, including the `Ether` class definition (`scapy/layers/l2.py:244`), the `dissect()` method (`scapy/packet.py:1049`), the `Padding` class (`scapy/packet.py:1906`), and the `ETHER_TYPES` data registry (`scapy/data.py:526–530`).


## 0.2 Documentation Discovery and Analysis


### 0.2.1 Existing Documentation Infrastructure Assessment

Repository analysis reveals a mature Sphinx-based documentation ecosystem with dedicated layer-specific reference pages, but **no existing documentation specifically covering Ethernet frame padding behavior or EtherType dispatch internals**.

**Documentation framework:** Sphinx, version `>=3.0.0` (declared in `pyproject.toml`)

**Documentation generator configuration:**
- Primary config: `doc/scapy/conf.py` — configures extensions (`autodoc`, `napoleon`, `todo`, `linkcode`, `scapy_doc`), import paths, and multi-format output (HTML, LaTeX, man, Texinfo)
- Build entry point: `doc/scapy/Makefile` (Unix) and `doc/scapy/make.bat` (Windows)
- CI build command: `sphinx-build -W --keep-going -b html . _build/html` (from `tox.ini`)
- ReadTheDocs config: `.readthedocs.yml` (Ubuntu 20.04, Python 3.9, EPUB+PDF outputs)

**Existing documentation structure:**

| Path | Type | Relevance |
|------|------|-----------|
| `doc/scapy/build_dissect.rst` | Layer development guide | **HIGH** — documents `dissect()`, `extract_padding()`, `bind_layers()`, `guess_payload_class()` from a protocol-developer perspective |
| `doc/scapy/usage.rst` | User tutorial | MEDIUM — covers general packet crafting and interactive usage |
| `doc/scapy/introduction.rst` | Overview | LOW — philosophy and quick demo |
| `doc/scapy/layers/` | Protocol references | MEDIUM — contains docs for specific layers (HTTP, TCP, Bluetooth, etc.) but **no Ethernet/L2 layer documentation** |
| `doc/notebooks/Scapy in 15 minutes.ipynb` | Notebook tutorial | MEDIUM — introductory walkthrough of packet construction |
| `README.md` | Project overview | LOW — general project description |
| `CONTRIBUTING.md` | Contributor guide | LOW — PR/testing workflow |

**Key finding:** The `doc/scapy/layers/` directory contains reference documentation for 11 protocol areas (automotive, bluetooth, http, kerberos, netflow, ntlm, pnio, sctp, tcp, tuntap) but does **not** include an Ethernet/L2 layer reference. The most relevant existing document is `doc/scapy/build_dissect.rst`, which explains the `extract_padding()` and `bind_layers()` mechanisms at a general level but does not address Ethernet-specific behaviors.

**Destination directory status:** The `blitzy/documentation/` directory does not yet exist and must be created.

### 0.2.2 Repository Code Analysis for Documentation

**Source code files examined for Ethernet/padding/EtherType behavior:**

| File | Key Content | Lines Examined |
|------|-------------|----------------|
| `scapy/layers/l2.py` | `Ether` class definition, `dispatch_hook()`, `bind_layers(Ether, ...)` calls, `ETHER_TYPES` additions | Lines 1–60, 240–330, 687–715 |
| `scapy/packet.py` | `Packet.dissect()`, `extract_padding()`, `do_dissect_payload()`, `guess_payload_class()`, `default_payload_class()`, `build()`, `build_padding()`, `Padding` class, `bind_layers()` / `bind_bottom_up()` / `bind_top_down()` definitions | Lines 730–810, 970–1100, 1760–1997 |
| `scapy/layers/inet.py` | `IP.extract_padding()` implementation | Found via `inspect.getsource()` |
| `scapy/data.py` | `ETHER_TYPES` loading from `/etc/ethertypes` or `scapy/libs/ethertypes.py` fallback, `EtherDA` class | Lines 320–540 |
| `scapy/libs/ethertypes.py` | Bundled EtherType database (fallback data); confirmed `0x9000` is **not** in the table | Full file scan |
| `scapy/config.py` | `conf.padding = 1` (line 787), `conf.padding_layer` (line 768), `conf.raw_layer` | Lines 768–800 |

**Key directories examined:**
- `scapy/layers/` — all 55+ protocol dissector modules
- `scapy/` — core package root (packet engine, configuration, utilities)
- `doc/scapy/` — Sphinx source tree for the published manual
- `doc/scapy/layers/` — protocol-specific reference pages
- `doc/notebooks/` — Jupyter tutorial notebooks

**Related documentation found:**
- `doc/scapy/build_dissect.rst` — provides background context for dissection pipeline and layer binding, useful as a reference style guide
- No existing documentation specifically covers the Ethernet layer's padding or EtherType dispatch behavior

### 0.2.3 Web Search Research Conducted

No external web search was required for this task. All investigation questions are answerable directly from the Scapy source code, which is the authoritative truth per the user's instructions. The codebase provides complete evidence for:
- Padding behavior (or lack thereof) during packet construction
- The `extract_padding()` mechanism for dissection-time padding detection
- The `bind_layers()` → `payload_guess` → `guess_payload_class()` → `default_payload_class()` fallback chain
- The `ETHER_TYPES` registry and its population from `/etc/ethertypes` or bundled data


## 0.3 Documentation Scope Analysis


### 0.3.1 Code-to-Documentation Mapping

The documentation deliverable consolidates findings from six interconnected source modules. Each module contributes specific insights to the user's questions.

**Module: `scapy/layers/l2.py` — Ethernet Layer Definition**
- Public APIs: `Ether` class (line 244), `Ether.dispatch_hook()` (line 267), `Dot3` class (line 275), `Dot3.extract_padding()` (line 281)
- Current documentation: `doc/scapy/build_dissect.rst` covers general binding concepts but **no dedicated Ethernet reference exists** in `doc/scapy/layers/`
- Documentation needed: Explanation of `Ether` field defaults (particularly `type=0x9000`), the `dispatch_hook` Ether-vs-Dot3 decision, and the absence of `extract_padding()` on `Ether`

**Module: `scapy/packet.py` — Packet Engine Core**
- Public APIs: `Packet.dissect()` (line 1049), `Packet.extract_padding()` (line 982), `Packet.guess_payload_class()` (line 1062), `Packet.default_payload_class()` (line 1081), `Packet.build()` (line 746), `Packet.build_padding()` (line 742), `Padding` class (line 1906), `bind_layers()` (line 1975), `bind_bottom_up()` (line 1931), `bind_top_down()` (line 1953)
- Current documentation: `doc/scapy/build_dissect.rst` describes the general dissection and building pipeline
- Documentation needed: Specific tracing of how these methods interact for Ethernet frames, with emphasis on the padding path and unknown-type fallback

**Module: `scapy/layers/inet.py` — IP Layer (extract_padding source)**
- Public APIs: `IP.extract_padding()` — uses `self.len - (self.ihl << 2)` to split payload from padding
- Current documentation: General coverage in `build_dissect.rst`
- Documentation needed: Explanation of how IP's `extract_padding()` is what actually identifies padding in Ethernet frames (not the Ether layer itself)

**Module: `scapy/data.py` — Protocol Registries**
- Public APIs: `ETHER_TYPES` (loaded at lines 526–530), `load_ethertypes()` (line 355), `EtherDA` class (line 331)
- Current documentation: None specific
- Documentation needed: Explanation of how `ETHER_TYPES` is populated and why `0x9000` is absent from the table

**Module: `scapy/libs/ethertypes.py` — Bundled EtherType Fallback Data**
- Content: The `DATA` bytestring containing the fallback EtherType table parsed from OpenBSD sources
- Documentation needed: Confirmation that `0x9000` is not present in this fallback table

**Module: `scapy/config.py` — Runtime Configuration**
- Public APIs: `conf.padding` (line 787, default `1`), `conf.padding_layer` (line 768), `conf.raw_layer`
- Documentation needed: Explanation of how `conf.padding` controls whether `Padding` layers appear during dissection

### 0.3.2 Documentation Gap Analysis

Given the requirements and repository analysis, documentation gaps include:

- **No Ethernet layer reference**: The `doc/scapy/layers/` directory covers 11 protocol areas but has no Ethernet/L2 page
- **Padding internals undocumented**: While `build_dissect.rst` mentions `extract_padding()`, it does not explain the per-layer behavior differences (e.g., `Ether` does not override it, `IP` does, `Dot3` does)
- **EtherType dispatch chain undocumented**: The full `bind_layers()` → `payload_guess` → `guess_payload_class()` → `default_payload_class()` → `conf.raw_layer` fallback is not traced end-to-end in any existing documentation
- **Build-time vs. dissection-time padding distinction undocumented**: No existing documentation explains that Scapy does **not** pad frames during `build()` but may detect padding during `dissect()`
- **The `0x9000` default EtherType is not explained**: The `Ether` class defaults to `type=0x9000` (Configuration Testing Protocol / Loopback), but this choice and its absence from `ETHER_TYPES` are not documented anywhere

All of these gaps will be addressed in the new `blitzy/documentation/scapy_0925ada48540.md` document.


## 0.4 Documentation Implementation Design


### 0.4.1 Documentation Structure Planning

The output is a single comprehensive markdown document. Its internal structure follows the user's question flow, moving from behavioral observations to source code analysis.

```
blitzy/
└── documentation/
    └── scapy_0925ada48540.md
        ├── Introduction & Methodology
        ├── Experiment 1: Normal Ethernet/IP/TCP Packet
        ├── Experiment 2: Short Payload Padding Behavior
        │   ├── Build-time: Does Scapy pad?
        │   ├── Dissection-time: raw() roundtrip
        │   └── Padding threshold discovery
        ├── Experiment 3: Unknown EtherType Behavior
        │   ├── 0x9000 (Ether default type)
        │   ├── 0xBEEF (arbitrary unknown)
        │   └── 0x1234 (arbitrary unknown)
        ├── Source Code Deep Dive
        │   ├── Where Scapy decides to add padding
        │   ├── How EtherType dispatch works
        │   └── What happens with unknown protocols
        └── Summary of Findings
```

### 0.4.2 Content Generation Strategy

**Information Extraction Approach:**
- Extract behavioral observations from 16 controlled in-memory experiments executed against the installed Scapy package
- Extract source code logic by reading `scapy/layers/l2.py`, `scapy/packet.py`, `scapy/layers/inet.py`, `scapy/data.py`, and `scapy/config.py`
- Cross-reference experiment results with source code to provide "why" explanations

**Documentation Standards:**
- Markdown formatting with proper headers (`#`, `##`, `###`)
- Code examples using ` ```python ` blocks with syntax highlighting
- Source citations as inline references: `Source: scapy/packet.py:1049`
- Tables for summarizing key findings and threshold data
- Mermaid diagrams for the dissection pipeline and EtherType dispatch flowchart
- Each answer section includes: question restatement, experimental code, observed output, source-code explanation, and conclusion

### 0.4.3 Diagram and Visual Strategy

**Mermaid diagrams to create within the document:**

- **Dissection Pipeline Flowchart**: Illustrates the flow from `Packet.dissect()` → `do_dissect()` → `extract_padding()` → `do_dissect_payload()` → `Padding` layer creation, showing where padding decisions happen
- **EtherType Dispatch Sequence**: Shows the `guess_payload_class()` → iterate `payload_guess` → `default_payload_class()` → `conf.raw_layer` fallback chain
- **Build vs. Dissect Comparison**: Side-by-side view of how `Packet.build()` (no padding logic) differs from `Packet.dissect()` (padding-aware via `extract_padding()`)

These diagrams will be embedded directly in the markdown file using ` ```mermaid ` code blocks.


## 0.5 Documentation File Transformation Mapping


### 0.5.1 File-by-File Documentation Plan

| Target Documentation File | Transformation | Source Code/Docs | Content/Changes |
|---------------------------|----------------|------------------|-----------------|
| `blitzy/documentation/scapy_0925ada48540.md` | CREATE | `scapy/layers/l2.py`, `scapy/packet.py`, `scapy/layers/inet.py`, `scapy/data.py`, `scapy/libs/ethertypes.py`, `scapy/config.py` | Comprehensive Q&A document covering Ethernet frame padding behavior, unknown EtherType handling, raw roundtrip fidelity, padding threshold analysis, and source code tracing with reproducible Python examples and Mermaid diagrams |

**Transformation Mode: CREATE** — This is the only file produced. No existing files are modified, updated, or deleted per the user's explicit read-only constraint and the `SWE-AtlasQnA-Repo` implementation rule.

### 0.5.2 New Documentation File Detail

```
File: blitzy/documentation/scapy_0925ada48540.md
Type: Technical Investigation / Q&A Reference
Source Code:
  - scapy/layers/l2.py (Ether class, dispatch_hook, bind_layers calls)
  - scapy/packet.py (dissect, extract_padding, guess_payload_class, build, build_padding, Padding class, bind_layers definition)
  - scapy/layers/inet.py (IP.extract_padding)
  - scapy/data.py (ETHER_TYPES loading, EtherDA class)
  - scapy/libs/ethertypes.py (bundled EtherType fallback data)
  - scapy/config.py (conf.padding, conf.padding_layer, conf.raw_layer)
Sections:
  - Introduction and Methodology
  - Experiment 1: Normal Ether/IP/TCP packet (build and inspect)
  - Experiment 2: Short payload padding behavior
    - Build-time observation (Scapy does NOT pad)
    - Dissection-time observation (IP.extract_padding detects padding)
    - Raw roundtrip behavior (Padding layer appears when NIC-level padding is present)
  - Experiment 3: Padding threshold discovery
    - Systematic size sweep from 0 to 60 bytes
    - Identification of the 46-byte payload threshold (14 + 46 = 60)
  - Experiment 4: Unknown EtherType behavior
    - 0x9000, 0xBEEF, 0x1234 tests
    - Fallback to Raw layer (no error, no guessing)
  - Source Code Deep Dive: Padding Logic
    - Packet.build() path (no padding injection)
    - Packet.dissect() path (extract_padding + conf.padding)
    - Padding class (scapy/packet.py:1906)
    - IP.extract_padding() as the actual padding detector
  - Source Code Deep Dive: EtherType Dispatch
    - bind_layers() mechanism
    - Ether.payload_guess list (16 registered bindings)
    - guess_payload_class() → default_payload_class() → conf.raw_layer
    - ETHER_TYPES registry (display-name lookup, not dispatch)
  - Source Code Deep Dive: Ether.dispatch_hook
    - Ether vs Dot3 selection based on type field ≤ 1500
  - Summary of Key Findings
Diagrams:
  - Dissection pipeline flowchart (Mermaid)
  - EtherType dispatch sequence (Mermaid)
Key Citations:
  - scapy/layers/l2.py:244 (Ether class)
  - scapy/packet.py:746 (build method)
  - scapy/packet.py:1049 (dissect method)
  - scapy/packet.py:1062 (guess_payload_class)
  - scapy/packet.py:1081 (default_payload_class)
  - scapy/packet.py:1906 (Padding class)
  - scapy/layers/inet.py (IP.extract_padding)
  - scapy/data.py:526 (ETHER_TYPES loading)
  - scapy/config.py:787 (conf.padding)
```

### 0.5.3 Documentation Configuration Updates

No documentation configuration updates are required. The output file is a standalone markdown document placed in `blitzy/documentation/`. It does not integrate with Scapy's existing Sphinx documentation infrastructure (`doc/scapy/`), as the user's intent is an independent investigative Q&A document, not a modification to the published Scapy manual.

### 0.5.4 Cross-Documentation Dependencies

- **No shared includes** — The output document is self-contained
- **No navigation links** — The document does not link to or from Scapy's Sphinx docs
- **No TOC updates** — No changes to `doc/scapy/index.rst` or any other navigation file
- **Internal cross-references** — The document internally cross-references source code paths (e.g., "see `scapy/packet.py:1049` for the dissect method") but creates no external dependencies


## 0.6 Dependency Inventory


### 0.6.1 Documentation Dependencies

The documentation task requires only the Scapy package itself to execute the investigative experiments. No additional documentation-generation tools (Sphinx, MkDocs, etc.) are needed since the output is a standalone markdown file.

| Registry | Package Name | Version | Purpose |
|----------|--------------|---------|---------|
| pip (local) | scapy | 2.7.0 (from repo) | Packet construction, serialization, and dissection experiments |
| pip | setuptools | >=62.0.0 | Build backend for editable install of Scapy from source |
| system | python3 | >=3.7, <4 (3.10 highest in classifiers; 3.11 highest in tox.ini) | Python runtime to execute Scapy and experimental scripts |

**Note on documentation tooling:** While the Scapy project uses Sphinx (`>=3.0.0`) with `sphinx_rtd_theme` (`>=0.4.3`) for its official documentation pipeline, these tools are **not** required for this task. The deliverable is a plain markdown file, not a Sphinx-rendered document.

### 0.6.2 Documentation Reference Updates

Not applicable. No existing documentation files require link updates, as the new document is placed in a separate `blitzy/documentation/` directory and has no inbound or outbound links to/from the existing Scapy documentation tree.


## 0.7 Coverage and Quality Targets


### 0.7.1 Documentation Coverage Metrics

**Current coverage analysis (before this task):**
- Ethernet layer padding behavior documented: 0% — no existing documentation covers this topic
- EtherType dispatch internals documented: ~20% — `doc/scapy/build_dissect.rst` describes `bind_layers()` and `guess_payload_class()` generically but does not trace the Ethernet-specific path
- Unknown EtherType handling documented: 0% — no existing documentation addresses the fallback to `Raw` for unrecognized EtherTypes
- Ether class field defaults documented: 0% — the default `type=0x9000` is not explained anywhere

**Target coverage (after this task): 100%** for all six user questions (R1–R6), achieved through the new `blitzy/documentation/scapy_0925ada48540.md` document.

| Topic | Before | After | Evidence Source |
|-------|--------|-------|-----------------|
| Normal Ether/IP/TCP construction | 0% | 100% | Experiment 1 with full output and explanation |
| Short-payload padding behavior | 0% | 100% | Experiments 2, 4, 5, 6 with code and traces |
| Raw roundtrip fidelity | 0% | 100% | Experiment 7 demonstrating Padding layer persistence |
| Unknown EtherType handling | 0% | 100% | Experiments 10, 11, 12 with dispatch chain trace |
| Padding threshold identification | 0% | 100% | Systematic sweep across 0–60 byte payloads |
| Source code tracing (padding + dispatch) | 0% | 100% | Line-level citations to l2.py, packet.py, inet.py, data.py, config.py |

### 0.7.2 Documentation Quality Criteria

**Completeness requirements:**
- Every user question has a dedicated section with: question restatement, experimental code, observed output, source-code explanation, and clear conclusion
- All source code citations include file path and line number
- Mermaid diagrams illustrate the dissection pipeline and EtherType dispatch chain

**Accuracy validation:**
- All code examples were executed against the installed Scapy package (version from repo, commit `scapy_0925ada48540`)
- All experiment outputs are captured from actual execution, not fabricated
- All source code citations were verified by reading the actual files via `read_file`

**Clarity standards:**
- Technical accuracy with accessible explanations for each behavioral observation
- Progressive disclosure: observations first, then source-code "why" explanations
- Clear separation between "what Scapy does" and "where in the code this happens"

**Maintainability:**
- Source citations use `file:line` format for easy future verification
- The document is self-contained and does not depend on external state

### 0.7.3 Example and Diagram Requirements

- **Minimum examples per question**: 1 reproducible Python code block with observed output
- **Diagram types required**: 2 Mermaid diagrams (dissection pipeline flowchart, EtherType dispatch sequence)
- **Code example testing**: All examples executed in the environment with Scapy installed from the local repository
- **Visual content freshness**: Diagrams derived from current source code structure at the examined commit


## 0.8 Scope Boundaries


### 0.8.1 Exhaustively In Scope

**New documentation files:**
- `blitzy/documentation/scapy_0925ada48540.md` — the sole deliverable

**Source code files to analyze (read-only):**
- `scapy/layers/l2.py` — Ether class definition, dispatch_hook, bind_layers calls
- `scapy/packet.py` — Packet base class: dissect(), extract_padding(), guess_payload_class(), default_payload_class(), build(), build_padding(), Padding class, bind_layers()/bind_bottom_up()/bind_top_down() functions
- `scapy/layers/inet.py` — IP.extract_padding() implementation
- `scapy/data.py` — ETHER_TYPES loading, EtherDA class
- `scapy/libs/ethertypes.py` — bundled EtherType fallback data
- `scapy/config.py` — conf.padding, conf.padding_layer, conf.raw_layer settings

**Topics to document:**
- Ethernet frame construction behavior (field defaults, byte serialization)
- Padding behavior during build (none) vs. dissection (via extract_padding)
- Raw-bytes roundtrip (`raw(pkt)` → `Ether(raw_bytes)`) with and without NIC-level padding
- Unknown EtherType handling (dispatch chain fallback to Raw)
- Padding threshold (60-byte minimum, 46-byte payload threshold for Ether-only)
- Source code location and logic for all observed behaviors

**Experimental activities (in-memory only):**
- Building test packets with `Ether()`, `IP()`, `TCP()`, `Raw()`
- Serializing packets via `raw()`
- Dissecting raw bytes via `Ether(raw_bytes)`
- Inspecting layer structure via `.show()`, `.haslayer()`, `.layers()`
- Reading class attributes (`payload_guess`, `ETHER_TYPES`)

### 0.8.2 Explicitly Out of Scope

- **Source code modifications**: No files in the Scapy repository may be modified, created, or deleted — per the user's explicit instruction and the `SWE-AtlasQnA-Repo` implementation rule
- **Existing documentation updates**: No changes to `doc/scapy/*.rst`, `README.md`, `CONTRIBUTING.md`, or any other existing documentation file
- **Network I/O**: No packets are sent or received on any network interface — all experiments are in-memory only
- **Sphinx build**: No documentation build is performed — the output is a standalone markdown file
- **Test file modifications**: No changes to `test/` directory contents
- **Non-Ethernet protocols**: While the document references IP and TCP in examples, the focus is exclusively on Ethernet-layer behavior; deep-dive documentation of IP or TCP internals is out of scope
- **Performance analysis**: No benchmarking or performance testing of Scapy's packet construction or dissection
- **Feature additions or code refactoring**: Strictly a documentation/investigation task


## 0.9 Execution Parameters


### 0.9.1 Documentation-Specific Instructions

- **Documentation build command**: Not applicable — the output is a standalone markdown file, not integrated into Sphinx
- **Documentation preview command**: Any markdown viewer or `cat blitzy/documentation/scapy_0925ada48540.md`
- **Diagram generation command**: Diagrams are embedded as Mermaid code blocks within the markdown; no external generation step is needed (renderers like GitHub, VS Code, or any Mermaid-compatible viewer will render them inline)
- **Documentation deployment command**: Not applicable — the file is committed directly to the repository
- **Default format**: Markdown with Mermaid diagrams
- **Citation requirement**: Every technical claim must reference the specific source file and line number
- **Style guide**: Follow the investigative Q&A format: question → experiment → observation → code trace → conclusion
- **Documentation validation**: Verify that all code examples produce the documented output when run against the installed Scapy package


## 0.10 Rules for Documentation


The following rules are derived from the user's explicit instructions and the `SWE-AtlasQnA-Repo` implementation rule:

- **Do not modify any existing files in the source repository.** The investigation is read-only; the only file created is the output markdown document.
- **Create a new markdown document named `scapy_0925ada48540.md`** (matching the `<source_branch_name>`) and place it in the `blitzy/documentation/` directory.
- **Provide thinking and rationale behind the answers.** Every behavioral observation must be explained by tracing the responsible source code logic, citing specific file paths and line numbers.
- **Do not make assumptions — base all answers on the code as the truth.** All conclusions must be derived from actual experiment results and verified source code reading, not from general knowledge about Ethernet standards or Scapy documentation.
- **Include reproducible code examples** for each behavioral observation so readers can verify findings independently.
- **Use Mermaid diagrams** to visualize the dissection pipeline and EtherType dispatch chain for clarity.
- **All experiments are in-memory only** — no network I/O or packet transmission.


## 0.11 References


### 0.11.1 Files and Folders Searched

The following files and folders were examined during the analysis phase to derive all conclusions documented in this Agent Action Plan:

**Core source files (read in detail):**

| File Path | Lines Examined | Key Content Retrieved |
|-----------|----------------|----------------------|
| `scapy/layers/l2.py` | 1–60, 240–330, 687–715 | Ether class definition (line 244), fields_desc with `type=0x9000` default, `dispatch_hook()` (line 267), all `bind_layers(Ether, ...)` calls, ETHER_TYPES additions (lines 240–241) |
| `scapy/packet.py` | 730–810, 970–1100, 1760–1997 | `build()` (line 746), `build_padding()` (line 742), `dissect()` (line 1049), `extract_padding()` (line 982), `do_dissect_payload()` (line 1023), `guess_payload_class()` (line 1062), `default_payload_class()` (line 1081), `Padding` class (line 1906), `Raw` class (line 1877), `bind_layers()` (line 1975), `bind_bottom_up()` (line 1931), `bind_top_down()` (line 1953), `NoPayload.build_padding()` (line 1770) |
| `scapy/layers/inet.py` | Extracted via `inspect.getsource()` | `IP.extract_padding()` — computes `tmp_len = self.len - (self.ihl << 2)` to split payload from padding |
| `scapy/data.py` | 320–540 | `EtherDA` class (line 331), `load_ethertypes()` (line 355), `ETHER_TYPES` initialization (lines 526–530) |
| `scapy/libs/ethertypes.py` | Full file (first 80 lines + grep for 0x9000) | Bundled EtherType fallback DATA; confirmed `0x9000` is absent |
| `scapy/config.py` | 768–800 | `conf.padding_layer` (line 768), `conf.padding = 1` (line 787) |
| `pyproject.toml` | Full file | Python version requirement `>=3.7, <4`, classifiers up to 3.10, docs extras (sphinx>=3.0.0, sphinx_rtd_theme>=0.4.3), setuptools build backend |
| `tox.ini` | First 80 lines | Test matrix (py27–py311), docs build command, apitree generation |
| `.readthedocs.yml` | Full file | ReadTheDocs config (Ubuntu 20.04, Python 3.9, EPUB/PDF output) |
| `README.md` | First 60 lines | Project overview, supported platforms, Python version note |
| `doc/scapy/build_dissect.rst` | Lines 280–575 | Dissection pipeline documentation, extract_padding explanation, bind_layers usage, guess_payload_class description, build pipeline |
| `doc/scapy/conf.py` | First 50 lines | Sphinx configuration: extensions, autodoc settings, version metadata |

**Folders explored:**

| Folder Path | Depth | Key Findings |
|-------------|-------|--------------|
| `` (root) | 1 | Project structure: scapy/, doc/, test/, .config/, .github/, build files |
| `scapy/` | 1 | Core package: 34 modules + 7 subpackages |
| `scapy/layers/` | 1 | 55+ protocol dissector modules, `tls/` subpackage |
| `doc/` | 1 | Documentation umbrella: notebooks/, scapy/ (Sphinx), syntax/, vagrant_ci/ |
| `doc/scapy/` | 1 | Sphinx source tree: 15 RST files, conf.py, Makefile, 4 subfolders |
| `doc/scapy/layers/` | 1 | 11 protocol reference RSTs (no Ethernet/L2 page) |

### 0.11.2 Attachments

No attachments were provided for this project. No Figma screens or external design files are applicable.

### 0.11.3 Experimental Evidence Summary

16 experiments were executed in-memory using the installed Scapy package to validate all behavioral claims:

| Experiment | Description | Key Finding |
|------------|-------------|-------------|
| 1 | Normal Ether/IP/TCP packet | 54 bytes total; fields auto-populated |
| 2 | Short payload (Ether/IP/Raw 10 bytes) | 44 bytes; no auto-padding to 60 |
| 2b | Roundtrip short packet | No Padding layer (no extra bytes in raw) |
| 3 | Unknown EtherType 0x9000 | Payload becomes Raw; no error |
| 3b | Unknown EtherType 0xBEEF | Payload becomes Raw; no error |
| 3c | Unknown EtherType 0x1234 | Payload becomes Raw; no error |
| 4 | Size sweep: Ether/Raw 0–60 bytes | No padding at any size; raw = 14 + payload exactly |
| 5 | Size sweep: Ether/IP/TCP/Raw 0–46 bytes | Threshold at TCP payload=6 (total=60) |
| 6 | Manual NIC-pad to 60 + dissect | Padding layer detected (16 bytes) via IP.extract_padding |
| 7 | Roundtrip with NIC-padded frame | Padding preserved; rebuilt raw = 60 bytes |
| 8 | conf.padding=0 test | Padding layer suppressed during dissection |
| 9 | IP.extract_padding source inspection | Uses `self.len - (self.ihl << 2)` |
| 10 | Ether.payload_guess inspection | 16 registered EtherType bindings |
| 11 | guess_payload_class / default_payload_class | Falls back to conf.raw_layer (Raw) |
| 12 | ETHER_TYPES lookup for various values | 0x9000, 0xBEEF, 0x1234 all NOT FOUND |
| 13 | Ether.dispatch_hook source | type ≤ 1500 → Dot3; else Ether |
| 14 | Ether has no extract_padding | Confirmed: not in Ether.__dict__ |
| 15 | Full build + NIC-pad + dissect analysis | IP.len drives padding detection |
| 16 | Ether-only (no IP) with padding | Null bytes merge into Raw.load; no Padding layer |


