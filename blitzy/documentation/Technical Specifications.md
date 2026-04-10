# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

### 0.1.1 Core Documentation Objective

Based on the provided requirements, the Blitzy platform understands that the documentation objective is to **create new documentation** that comprehensively answers a set of deep, code-grounded questions about Scapy's packet dissection mechanism—specifically, how raw bytes are transformed into a nested, typed layer hierarchy at runtime.

- **Documentation Category:** Create new documentation
- **Documentation Type:** Technical Q&A / Investigative deep-dive
- **Target Artifact:** A standalone Markdown document named `scapy_0925ada48540.md` placed in the `blitzy/documentation/` directory of the destination repository

The user is onboarding into the Scapy codebase and has articulated the following concrete lines of inquiry, each requiring code-grounded answers with runtime evidence:

- **Layer boundary discovery:** How does Scapy determine where one protocol layer ends and the next begins (e.g., Ethernet → IP → TCP) when dissecting raw bytes?
- **Runtime dissection walkthrough:** What happens step-by-step when a real packet (such as an HTTP request over TCP/IP/Ethernet) is dissected? The user wants to see the dissection path and the resulting nested structure at each level.
- **Unknown / unrecognized payloads:** What does Scapy do when there is no registered next-layer binding for the remaining bytes—does it "give up gracefully," and what does the resulting packet object look like?
- **Trust model for header fields:** If a crafted packet has an Ethernet type field claiming IPv6 (`0x86dd`) but the actual payload bytes contain an IPv4 header, does Scapy follow the header blindly or detect the mismatch?
- **Tunneled / encapsulated traffic:** For tunneled traffic such as GRE encapsulating another IP packet, does the dissection recurse all the way through the inner layers, and what does that structure look like?

### 0.1.2 Special Instructions and Constraints

The user has specified the following critical directives:

- **Read-only repository:** The source repository itself must remain unchanged. No existing files may be modified.
- **Temporary scripts permitted:** Observation scripts may be used during analysis but must be cleaned up afterward.
- **Code-as-truth:** All answers must be grounded in the actual source code, not assumptions or general knowledge.
- **Thinking / rationale required:** The generated document must include the reasoning behind each answer.
- **Placement rule:** The output document goes in `blitzy/documentation/scapy_0925ada48540.md`.

Implementation rules from the project configuration:

- *"Create a new markdown document named `<source_branch_name>.md` that comprehensively answers the question(s) posed in the prompt."*
- *"Provide thinking / rationale behind the answers."*
- *"Do not make assumptions, base your answers on the code as the truth."*
- *"Do not modify any existing files in the source repository."*
- *"Place the generated document in the `blitzy/documentation` directory in the destination repo."*

No user-provided templates, style guides, or Figma assets are attached.

### 0.1.3 Technical Interpretation

These documentation requirements translate to the following technical documentation strategy:

- To **explain layer boundary discovery**, we will trace the `bind_layers()` registrations in `scapy/packet.py` (lines 1975–1996), the `payload_guess` list on each `Packet` subclass, and the runtime path through `guess_payload_class()` (line 1062) and `do_dissect_payload()` (line 1023).
- To **walk through HTTP request dissection**, we will construct a synthetic Ethernet/IP/TCP/HTTP byte stream, feed it to `Ether()`, and document the recursive dissection chain: `Ether.dissect()` → `IP.dissect()` → `TCP.dissect()` → `HTTP.guess_payload_class()`, citing the specific `bind_layers()` calls in `scapy/layers/inet.py` (lines 1101–1115) and `scapy/layers/http.py` (lines 748–753).
- To **demonstrate unknown payload handling**, we will show that when `guess_payload_class()` exhausts the `payload_guess` list without a match, it calls `default_payload_class()` (line 1081) which returns `conf.raw_layer` (i.e., the `Raw` class defined at line 1877), and the remaining bytes are encapsulated as `Raw.load`.
- To **explore the trust model**, we will craft a packet with `Ether(type=0x86dd)` (IPv6 EtherType) but attach actual IPv4 bytes as payload, and show that Scapy follows the header's declared type field blindly—instantiating `IPv6()` on the bytes—without cross-checking whether the payload is actually valid IPv6.
- To **demonstrate GRE tunnel recursion**, we will construct an `Ether/IP/GRE/IP/TCP` packet and show that dissection recurses through every layer, including the inner IP and TCP, because `bind_layers(GRE, IP, proto=2048)` is registered in `scapy/layers/inet.py` (line 1103).

### 0.1.4 Inferred Documentation Needs

Based on code analysis, the following additional documentation elements are inferred:

- **Binding registry visualization:** The `payload_guess` list for `Ether`, `IP`, `TCP`, and `GRE` should be enumerated to show the complete binding table that drives dissection decisions.
- **Dispatch hook mechanism:** The `Ether.dispatch_hook()` method (in `scapy/layers/l2.py`, line 267) dynamically selects between `Ether` and `Dot3` based on the first two bytes of the type/length field—this is a dissection-time hook that precedes `bind_layers()` and should be documented.
- **Error handling path:** The `do_dissect_payload()` method wraps payload instantiation in a try/except, and on failure consults `conf.debug_dissector` to decide between silent fallback, logging, or re-raising (lines 1037–1046). This graceful degradation is relevant to the "unknown payload" question.
- **HTTP layer loading nuance:** The HTTP layer (`scapy/layers/http.py`) is **not** in the default `load_layers` list (`scapy/config.py`, line 847 comment), meaning TCP port-80 traffic will not be recognized as HTTP unless `load_layer('http')` is explicitly called—an important operational detail for the user.
- **Mermaid diagrams:** The dissection control flow and the binding decision tree should be visualized with Mermaid diagrams to aid comprehension.


## 0.2 Documentation Discovery and Analysis

### 0.2.1 Existing Documentation Infrastructure Assessment

Repository analysis reveals a mature Sphinx-based documentation ecosystem with strong coverage of protocol extension guides but limited deep-dive material on the runtime dissection pipeline. The user's questions are partially addressed by existing docs but require significantly deeper treatment with runtime evidence.

- **Documentation framework:** Sphinx `>=3.0.0` with `sphinx_rtd_theme>=0.4.3`
- **Documentation generator configuration:** `doc/scapy/conf.py` (Sphinx configuration)
- **Hosted documentation:** ReadTheDocs at `https://scapy.readthedocs.io`, configured via `.readthedocs.yml` (Ubuntu 20.04, Python 3.9, EPUB/PDF outputs)
- **API documentation tools:** `sphinx.ext.autodoc`, `sphinx.ext.napoleon`, custom `scapy_doc` extension in `doc/scapy/_ext/`
- **Diagram tools detected:** None natively configured; Mermaid will be used in the new document
- **Build command:** `sphinx-build -W --keep-going -b html . _build/html` (from CI `tox.ini` docs job)

**Existing documentation files examined:**

| File / Path | Type | Relevance |
|---|---|---|
| `doc/scapy/build_dissect.rst` (1163 lines) | Sphinx RST | **High** — Explains dissect lifecycle, `do_dissect()`, `guess_payload_class()`, `bind_layers()`, `payload_guess`, and building. Provides the closest existing coverage to the user's questions but lacks runtime evidence, trust-model analysis, and tunnel recursion detail. |
| `doc/scapy/usage.rst` | Sphinx RST | **Medium** — Covers interactive usage, session filtering, and `sniff()` sessions. Mentions dissection performance tuning and layer filtering. |
| `doc/scapy/extending.rst` | Sphinx RST | **Low** — Focuses on adding custom protocols, not on explaining the existing dissection engine. |
| `doc/notebooks/Scapy in 15 minutes.ipynb` | Jupyter Notebook | **Medium** — Introductory walkthrough covering packet construction and layer stacking, but no deep dissection trace. |
| `README.md` | Project overview | **Low** — High-level project description; no dissection detail. |
| `CONTRIBUTING.md` | Contributor guide | **Low** — Pull request and code style guidance. |

### 0.2.2 Repository Code Analysis for Documentation

The following source modules were inspected to extract the technical details needed to answer the user's questions:

**Core dissection engine:**
- `scapy/packet.py` — `Packet` class (line 77), `dissect()` (line 1049), `do_dissect()` (line 1002), `do_dissect_payload()` (line 1023), `guess_payload_class()` (line 1062), `default_payload_class()` (line 1081), `add_payload()` (line 360), `extract_padding()` (line 982), `Raw` (line 1877), `Padding` (line 1906), `NoPayload` (line 1696), `bind_layers()` (line 1975), `bind_bottom_up()` (line 1931), `bind_top_down()` (line 1953)
- `scapy/base_classes.py` — `Packet_metaclass` (line 281), `aliastypes` construction (line 358)
- `scapy/config.py` — `conf.raw_layer = Raw` (set at line 1921 of packet.py), `conf.debug_dissector`, `load_layers` list (lines 848–897)

**Protocol layer definitions and bindings:**
- `scapy/layers/l2.py` — `Ether` class (line 244) with `fields_desc` = `[dst, src, type]`, `dispatch_hook()` (line 267), `GRE` class (line 578), all `bind_layers()` calls (lines 686–716)
- `scapy/layers/inet.py` — `IP` class (line 521), `TCP` class (line 753), `UDP` class (line 834), `extract_padding()` for IP (line 553), all `bind_layers()` calls (lines 1101–1115) including `Ether→IP (type=2048)`, `IP→TCP (proto=6)`, `IP→GRE (proto=47)`, `GRE→IP (proto=2048)`
- `scapy/layers/inet6.py` — `bind_layers(Ether, IPv6, type=0x86dd)` (line 4083), `bind_layers(GRE, IPv6, proto=0x86dd)` (line 4085)
- `scapy/layers/http.py` — `HTTP` class (line 548), `HTTP.guess_payload_class()` (line 637) using regex for request/response detection, `bind_bottom_up(TCP, HTTP, sport=80)` (line 748), `bind_bottom_up(TCP, HTTP, dport=80)` (line 749), `bind_layers(TCP, HTTP, sport=80, dport=80)` (line 750)

**Key directories examined:**
- `scapy/` — Core package root
- `scapy/layers/` — All 48 default protocol modules
- `doc/scapy/` — Sphinx source tree
- `doc/notebooks/` — Jupyter tutorials

### 0.2.3 Web Search Research Conducted

No web searches were required for this task. All answers are grounded in the source code, per the user's explicit directive: *"Do not make assumptions, base your answers on the code as the truth."* The existing codebase provides complete evidence for every question posed.


## 0.3 Documentation Scope Analysis

### 0.3.1 Code-to-Documentation Mapping

The following source modules require documentation extraction and analysis to answer the user's five core questions:

- **Module:** `scapy/packet.py` (2553 lines)
  - Public APIs: `Packet.dissect()`, `Packet.do_dissect()`, `Packet.do_dissect_payload()`, `Packet.guess_payload_class()`, `Packet.default_payload_class()`, `Packet.extract_padding()`, `Packet.add_payload()`, `bind_layers()`, `bind_bottom_up()`, `bind_top_down()`
  - Sentinel classes: `Raw`, `Padding`, `NoPayload`
  - Current documentation: Partially covered in `doc/scapy/build_dissect.rst` but without runtime trace evidence
  - Documentation needed: Runtime walkthrough, error-handling path, trust-model analysis, unknown-payload behavior

- **Module:** `scapy/base_classes.py`
  - Public APIs: `Packet_metaclass.__new__()`, `aliastypes` construction
  - Current documentation: Not directly covered in user-facing docs
  - Documentation needed: How metaclass wires up `payload_guess` and `aliastypes` at class creation time

- **Module:** `scapy/layers/l2.py`
  - Protocol classes: `Ether` (3 fields: dst, src, type), `GRE` (proto field, conditional fields), `GRE_PPTP`
  - Binding registrations: `Ether→IP (type=2048)`, `Ether→IPv6 (type=0x86dd)`, `Ether→ARP (type=2054)`, `GRE→IP (proto=2048)`, `GRE→Ether (proto=0x6558)`, plus 10 more
  - `dispatch_hook()`: Ether/Dot3 selection based on type/length field threshold (1500)
  - Documentation needed: Binding table enumeration, dispatch_hook explanation

- **Module:** `scapy/layers/inet.py`
  - Protocol classes: `IP` (with `extract_padding()` using `len` and `ihl`), `TCP`, `UDP`, `ICMP`
  - Binding registrations: `Ether→IP`, `IP→TCP (proto=6)`, `IP→UDP (proto=17)`, `IP→GRE (proto=47)`, `IP→ICMP (proto=1)`, `GRE→IP (proto=2048)`
  - Documentation needed: Full dissection trace from IP through TCP

- **Module:** `scapy/layers/inet6.py`
  - Binding registrations: `Ether→IPv6 (type=0x86dd)`, `GRE→IPv6 (proto=0x86dd)`
  - Documentation needed: How the trust model uses this binding when type field is faked

- **Module:** `scapy/layers/http.py`
  - `HTTP.guess_payload_class()`: Regex-based detection of HTTP request vs response vs unknown
  - Binding: `bind_bottom_up(TCP, HTTP, sport=80)` and `bind_bottom_up(TCP, HTTP, dport=80)`
  - Not in default `load_layers`: requires explicit `load_layer('http')`
  - Documentation needed: HTTP layer loading caveat, regex-based dissection

- **Module:** `scapy/config.py`
  - `conf.raw_layer`, `conf.padding_layer`, `conf.debug_dissector`, `load_layers` list
  - Documentation needed: Default fallback configuration, debug_dissector modes

### 0.3.2 Documentation Gap Analysis

Given the requirements and repository analysis, the following documentation gaps are being filled by this task:

- **No existing runtime dissection trace:** `doc/scapy/build_dissect.rst` explains the mechanism conceptually but never shows a real packet being dissected layer-by-layer with actual byte offsets and field values.
- **Trust model undocumented:** No existing documentation discusses what happens when header fields claim one protocol but the actual bytes belong to another. The codebase reveals that Scapy trusts header fields unconditionally.
- **Unknown payload fallback not illustrated:** While `build_dissect.rst` mentions `Raw` as the default, it does not show what a packet object looks like when dissection falls through to `Raw`.
- **GRE tunnel recursion not demonstrated:** No existing documentation traces a multi-layer encapsulated packet through the full recursive dissection chain.
- **HTTP layer loading caveat not highlighted:** The fact that HTTP is not loaded by default (it is explicitly excluded from `load_layers` per `scapy/config.py` line 847 comment) and requires `load_layer('http')` is not prominently documented in any user-facing guide.
- **`dispatch_hook()` not connected to dissection narrative:** The Ether/Dot3 decision hook that fires before `bind_layers()` matching is not explained in the context of a full dissection trace.


## 0.4 Documentation Implementation Design

### 0.4.1 Documentation Structure Planning

The output document `blitzy/documentation/scapy_0925ada48540.md` will follow this structure, organized as a single self-contained Markdown file answering each user question as a major section:

```
blitzy/
└── documentation/
    └── scapy_0925ada48540.md
        ├── Introduction (context + methodology)
        ├── 1. How Layer Boundaries Are Determined
        │   ├── The bind_layers() Registration System
        │   ├── The payload_guess List
        │   ├── guess_payload_class() Runtime Logic
        │   ├── dispatch_hook() — Pre-Binding Class Selection
        │   └── Binding Tables for Ether, IP, TCP, GRE
        ├── 2. Runtime Dissection of an HTTP Request Packet
        │   ├── Constructing the Test Packet
        │   ├── Step-by-Step Dissection Trace
        │   ├── Ether.dissect() → do_dissect() → field extraction
        │   ├── IP dissection with extract_padding()
        │   ├── TCP dissection and port-based HTTP binding
        │   ├── HTTP.guess_payload_class() regex matching
        │   └── The load_layer('http') Caveat
        ├── 3. Unknown / Unrecognized Payloads
        │   ├── How default_payload_class() Returns Raw
        │   ├── What the Packet Object Looks Like
        │   └── Error Handling in do_dissect_payload()
        ├── 4. Trust Model — Mismatched Header Fields
        │   ├── Scapy Trusts Headers Unconditionally
        │   ├── Demonstrating with IPv6 EtherType + IPv4 Bytes
        │   └── Implications and Rationale
        ├── 5. Tunneled Traffic — GRE Recursion
        │   ├── GRE Binding Registrations
        │   ├── Full Dissection of Ether/IP/GRE/IP/TCP
        │   └── Recursive Depth and Structure
        └── Conclusion
```

### 0.4.2 Content Generation Strategy

**Information Extraction Approach:**
- Extract dissection method signatures and control flow from `scapy/packet.py` using direct source reading
- Extract binding registrations from `scapy/layers/l2.py`, `scapy/layers/inet.py`, `scapy/layers/inet6.py`, and `scapy/layers/http.py` by reading `bind_layers()` call sites
- Generate runtime examples by constructing packets programmatically with `Ether()/IP()/TCP()` and feeding raw bytes back through `Ether(raw_bytes)` to demonstrate the dissection path
- Create Mermaid diagrams by mapping the control flow in `dissect()` → `do_dissect()` → `extract_padding()` → `do_dissect_payload()` → `guess_payload_class()`

**Documentation Standards:**
- Markdown formatting with proper headers (`#`, `##`, `###`)
- Mermaid diagram integration using triple-backtick `mermaid` blocks for dissection control flow and binding decision trees
- Code examples using triple-backtick `python` blocks with concise, runnable snippets
- Source citations as inline references: `Source: scapy/packet.py:1062`
- Tables for binding registrations and field mappings

### 0.4.3 Diagram and Visual Strategy

The following Mermaid diagrams will be created for the output document:

- **Dissection lifecycle flowchart:** The full `dissect()` → `do_dissect()` → `extract_padding()` → `do_dissect_payload()` → `guess_payload_class()` → payload instantiation pipeline
- **Binding decision tree:** How `guess_payload_class()` iterates `payload_guess`, falls through to `default_payload_class()`, and returns `Raw` when no match is found
- **HTTP request dissection sequence diagram:** A layer-by-layer trace showing Ether → IP → TCP → HTTP → HTTPRequest with byte ranges and field values at each step
- **GRE tunnel recursion diagram:** Showing the nested dissection through Ether → IP → GRE → IP → TCP with the recursive re-entry into `dissect()`
- **Trust model illustration:** Showing the blind type-field lookup path that causes Scapy to instantiate IPv6 on IPv4 bytes


## 0.5 Documentation File Transformation Mapping

### 0.5.1 File-by-File Documentation Plan

| Target Documentation File | Transformation | Source Code/Docs | Content/Changes |
|---|---|---|---|
| `blitzy/documentation/scapy_0925ada48540.md` | CREATE | `scapy/packet.py`, `scapy/base_classes.py`, `scapy/layers/l2.py`, `scapy/layers/inet.py`, `scapy/layers/inet6.py`, `scapy/layers/http.py`, `scapy/config.py`, `doc/scapy/build_dissect.rst` | Comprehensive Q&A document answering all five user questions about packet dissection mechanics, with code-grounded explanations, runtime evidence, Mermaid diagrams, and thinking/rationale for each answer |

No other documentation files are created, updated, or deleted. The implementation rule explicitly states *"Do not modify any existing files in the source repository."*

### 0.5.2 New Documentation File Detail

```
File: blitzy/documentation/scapy_0925ada48540.md
Type: Technical Q&A / Investigative Deep-Dive
Source Code:
    - scapy/packet.py (lines 77–2058)
    - scapy/base_classes.py (lines 281–370)
    - scapy/layers/l2.py (lines 244–716)
    - scapy/layers/inet.py (lines 521–1130)
    - scapy/layers/inet6.py (lines 4083–4100)
    - scapy/layers/http.py (lines 425–754)
    - scapy/config.py (lines 847–897)
Sections:
    - Introduction (methodology, scope)
    - Question 1: How Layer Boundaries Are Determined
        - bind_layers() mechanism from packet.py:1975
        - payload_guess list from packet.py:104
        - guess_payload_class() from packet.py:1062
        - dispatch_hook() from l2.py:267
        - Binding tables for Ether, IP, TCP, GRE
    - Question 2: Runtime Dissection of an HTTP Request
        - Synthetic packet construction
        - Step-by-step trace: Ether → IP → TCP → HTTP → HTTPRequest
        - HTTP layer loading caveat (config.py line 847)
    - Question 3: Unknown / Unrecognized Payloads
        - default_payload_class() returns conf.raw_layer (Raw)
        - Resulting packet structure with Raw.load
        - Error handling via conf.debug_dissector
    - Question 4: Trust Model (Mismatched Header Fields)
        - Blind type-field lookup in guess_payload_class()
        - Runtime demonstration: Ether(type=0x86dd) + IPv4 bytes
        - Rationale: Scapy is a tool, not a validator
    - Question 5: GRE Tunnel Recursion
        - GRE binding registrations: GRE→IP, GRE→IPv6, GRE→Ether
        - Full recursive dissection of Ether/IP/GRE/IP/TCP
        - Nested structure evidence
    - Conclusion
Diagrams:
    - Dissection lifecycle flowchart (Mermaid)
    - Binding decision tree (Mermaid)
    - HTTP dissection sequence (Mermaid)
    - GRE tunnel recursion visualization (Mermaid)
Key Citations:
    - scapy/packet.py (primary: dissect engine, bind_layers, Raw, NoPayload)
    - scapy/layers/l2.py (Ether, GRE, dispatch_hook)
    - scapy/layers/inet.py (IP, TCP, binding registrations)
    - scapy/layers/inet6.py (IPv6 binding)
    - scapy/layers/http.py (HTTP, guess_payload_class, port bindings)
    - scapy/config.py (load_layers, conf.raw_layer)
    - doc/scapy/build_dissect.rst (existing conceptual documentation, cross-reference)
```

### 0.5.3 Documentation Configuration Updates

No documentation configuration files (e.g., `mkdocs.yml`, `docusaurus.config.js`, `.readthedocs.yml`, `doc/scapy/conf.py`) require updates. The new file is placed in `blitzy/documentation/`, which is a standalone output directory outside the Sphinx documentation tree. The existing Sphinx-based docs at `doc/scapy/` are not modified per the implementation rules.

### 0.5.4 Cross-Documentation Dependencies

- **Shared content:** None — the new document is self-contained
- **Navigation links:** Not required — `blitzy/documentation/` is independent of the Sphinx `doc/scapy/` tree
- **Cross-references:** The new document will reference `doc/scapy/build_dissect.rst` as prior art for readers who want the existing upstream documentation perspective
- **Index/glossary updates:** Not required


## 0.6 Dependency Inventory

### 0.6.1 Documentation Dependencies

The following tools and packages are relevant to this documentation exercise:

| Registry | Package Name | Version | Purpose |
|---|---|---|---|
| PyPI | scapy | 2.7.0 (from repo HEAD) | The subject of documentation; used to run observation scripts and validate runtime behavior |
| PyPI | sphinx | >=3.0.0 | Existing documentation build system (not modified in this task) |
| PyPI | sphinx_rtd_theme | >=0.4.3 | Existing documentation theme (not modified in this task) |
| stdlib | Python | >=3.7, <4 | Runtime for Scapy and any temporary observation scripts |

No additional documentation tools need to be installed. The output is a standalone Markdown file that does not depend on any documentation generator. Mermaid diagrams embedded in the Markdown are rendered by GitHub, GitLab, or any Mermaid-compatible viewer.

### 0.6.2 Runtime Dependencies for Observation Scripts

Temporary observation scripts (used during analysis, cleaned up afterward) rely only on Scapy itself:

| Dependency | Source | Purpose |
|---|---|---|
| `scapy.all` | Repository `scapy/` package | Construct synthetic packets and trace dissection behavior |
| `scapy.layers.http` | `scapy/layers/http.py` | Explicitly loaded via `load_layer('http')` to test HTTP dissection |
| `scapy.layers.l2` | `scapy/layers/l2.py` | Ether, GRE class access |
| `scapy.layers.inet` | `scapy/layers/inet.py` | IP, TCP class access |
| `scapy.layers.inet6` | `scapy/layers/inet6.py` | IPv6 class access for trust-model experiment |

No external network access, root privileges, or capture hardware is required. All experiments use synthetically constructed byte arrays fed directly into Scapy's `Packet()` constructor.

### 0.6.3 Documentation Reference Updates

Not applicable. No existing documentation files contain links that need updating, as the new document is placed in a standalone `blitzy/documentation/` directory and does not alter the Sphinx documentation tree or any existing cross-references.


## 0.7 Coverage and Quality Targets

### 0.7.1 Documentation Coverage Metrics

The user has posed five distinct questions. Coverage is measured as the number of questions fully answered with code-grounded evidence:

| Question | Topic | Source Modules | Coverage Target |
|---|---|---|---|
| Q1 | Layer boundary discovery | `scapy/packet.py`, `scapy/layers/l2.py`, `scapy/layers/inet.py` | 100% — must explain `bind_layers()`, `payload_guess`, `guess_payload_class()`, and `dispatch_hook()` with code citations |
| Q2 | Runtime HTTP request dissection | `scapy/layers/l2.py`, `scapy/layers/inet.py`, `scapy/layers/http.py` | 100% — must trace byte-level dissection through Ether→IP→TCP→HTTP with field values |
| Q3 | Unknown payload handling | `scapy/packet.py` (`Raw`, `default_payload_class()`) | 100% — must show the fallback path and the resulting `Raw` packet object |
| Q4 | Trust model (mismatched headers) | `scapy/packet.py`, `scapy/layers/inet6.py` | 100% — must demonstrate blind header-field trust with runtime output |
| Q5 | GRE tunnel recursion | `scapy/layers/l2.py`, `scapy/layers/inet.py` | 100% — must show full recursive dissection through GRE tunnel |

- **Target question coverage:** 5/5 (100%)
- **Source code citation requirement:** Every technical claim must reference a specific file and line number
- **Runtime evidence requirement:** Every question must be accompanied by runtime output (packet `show()` or `repr()`)

### 0.7.2 Documentation Quality Criteria

**Completeness requirements:**
- All five questions answered with full code-path explanations
- Each answer includes: thinking/rationale, source code citations, runtime demonstration, and a summary
- Binding tables for Ether, IP, TCP, and GRE enumerated completely
- The HTTP layer loading caveat explicitly documented

**Accuracy validation:**
- All code references verified against the actual repository source (no stale line numbers)
- Runtime examples validated by executing Scapy in the development environment
- `payload_guess` lists verified by programmatic enumeration

**Clarity standards:**
- Progressive disclosure: start with high-level concept, then drill into code
- Consistent terminology: "dissection" (not "parsing"), "binding" (not "linking"), "layer" (not "protocol header")
- Mermaid diagrams for complex control flows
- Tables for structured data (binding registrations, field values)

**Maintainability:**
- Source citations use `file:line` format for traceability
- No hardcoded byte values without explaining what they represent
- Self-contained document requiring no external context

### 0.7.3 Example and Diagram Requirements

- **Minimum runtime demonstrations per question:** 1 (with `show()` or `repr()` output)
- **Diagram types required:**
  - Flowchart: Dissection lifecycle (1 diagram)
  - Flowchart: Binding decision tree / `guess_payload_class()` logic (1 diagram)
  - Sequence diagram or flowchart: HTTP packet dissection trace (1 diagram)
  - Flowchart: GRE tunnel recursive dissection (1 diagram)
- **Total diagrams:** At least 4 Mermaid diagrams
- **Code example testing:** All Python snippets validated against the repository's Scapy installation


## 0.8 Scope Boundaries

### 0.8.1 Exhaustively In Scope

- **New documentation files:**
  - `blitzy/documentation/scapy_0925ada48540.md` — The sole deliverable: a comprehensive Markdown Q&A document

- **Source modules analyzed for documentation content:**
  - `scapy/packet.py` — Dissection engine, binding system, Raw/Padding/NoPayload sentinel classes
  - `scapy/base_classes.py` — `Packet_metaclass`, `aliastypes` wiring
  - `scapy/fields.py` — Field `getfield()` API used during `do_dissect()`
  - `scapy/layers/l2.py` — Ether, GRE, dispatch_hook, all bind_layers calls
  - `scapy/layers/inet.py` — IP, TCP, UDP, ICMP, all bind_layers calls, extract_padding
  - `scapy/layers/inet6.py` — IPv6, IPv6-specific bind_layers calls
  - `scapy/layers/http.py` — HTTP, guess_payload_class, port bindings
  - `scapy/config.py` — conf.raw_layer, conf.debug_dissector, load_layers list

- **Documentation topics covered:**
  - The `bind_layers()` / `payload_guess` registration and lookup mechanism
  - The `dissect()` → `do_dissect()` → `extract_padding()` → `do_dissect_payload()` → `guess_payload_class()` pipeline
  - The `dispatch_hook()` pre-binding class selection (Ether vs Dot3)
  - Runtime dissection trace of an HTTP request packet (Ether/IP/TCP/HTTP)
  - Unknown payload fallback to `conf.raw_layer` (Raw class)
  - Error handling and `conf.debug_dissector` behavior
  - Trust model: header-field-driven payload class selection without content validation
  - GRE tunnel recursive dissection
  - HTTP layer loading requirement (`load_layer('http')`)

- **Temporary artifacts (created and cleaned up):**
  - Observation scripts used during analysis to validate runtime behavior

### 0.8.2 Explicitly Out of Scope

- **Source code modifications:** No changes to any file in the repository. This is explicitly mandated by the user: *"the repository itself should remain unchanged."*
- **Test file modifications:** No test files are created or modified
- **Feature additions or code refactoring:** Not applicable; this is purely a documentation task
- **Deployment configuration changes:** No changes to `.readthedocs.yml`, `doc/scapy/conf.py`, `tox.ini`, or any CI/CD configuration
- **Existing documentation updates:** No modifications to `doc/scapy/build_dissect.rst`, `doc/scapy/usage.rst`, `README.md`, or any other existing documentation file
- **Sphinx documentation tree changes:** No updates to `doc/scapy/index.rst` toctree or navigation
- **Live network capture or packet injection:** All examples use synthetically constructed byte arrays; no root privileges or network interfaces required
- **Protocol layers beyond the user's questions:** The document focuses on Ether, IP, TCP, HTTP, GRE, IPv6, and Raw. Other protocols (DHCP, DNS, TLS, Bluetooth, etc.) are not in scope
- **Packet building documentation:** The user's questions focus on dissection (bytes → structured object), not building (structured object → bytes)
- **Performance analysis:** Dissection performance, caching (`raw_packet_cache`), and optimization strategies are not in scope


## 0.9 Execution Parameters

### 0.9.1 Documentation-Specific Instructions

- **Documentation build command:** Not applicable — the output is a standalone Markdown file, not part of the Sphinx build
- **Documentation preview command:** Any Markdown renderer (e.g., `grip`, GitHub preview, VS Code Markdown preview)
- **Diagram generation command:** Mermaid diagrams are embedded inline in the Markdown and rendered by GitHub/GitLab natively; no separate generation step required
- **Documentation deployment command:** Not applicable — the file is committed to `blitzy/documentation/` in the destination repository
- **Default format:** GitHub-Flavored Markdown with Mermaid diagram blocks
- **Citation requirement:** Every section must reference source files using `Source: path/to/file.py:LineNumber` format
- **Style guide:** Answers structured as: Thinking/Rationale → Code Evidence → Runtime Demonstration → Summary
- **Documentation validation:** Manual review; no automated linting or link-checking required for a standalone Q&A document

### 0.9.2 Observation Script Execution Parameters

Temporary Python scripts used during analysis to validate runtime behavior will follow these rules:

- **Execution environment:** `python3` from the repository root, with `PYTHONPATH` including the repo so that `scapy` is importable
- **No network access required:** All packets are constructed synthetically using `Ether()/IP()/TCP()` or raw `bytes()` objects
- **No root privileges required:** No `send()`, `sniff()`, or socket operations are used
- **Cleanup mandate:** Any temporary script files are deleted after use; no artifacts remain in the repository
- **Output capture:** Script output (packet `show()`, `repr()`, `ls()`) is captured and embedded in the documentation as code blocks


## 0.10 Rules for Documentation

The following rules are explicitly mandated by the user and the project configuration, and must be strictly observed:

- **Do not modify any existing files in the source repository.** The Scapy codebase is read-only for this task. No source code, tests, configuration, or documentation files in the existing repository tree may be altered.
- **Base all answers on the code as the truth.** Every claim, explanation, and runtime demonstration must be grounded in the actual Scapy source code. No assumptions, generalizations from external documentation, or "typical behavior" statements are acceptable without code evidence.
- **Provide thinking / rationale behind the answers.** Each answer must explain *why* the code behaves the way it does, not just *what* it does. The reasoning must connect the user's question to specific code paths and design decisions.
- **Create the output document as `<source_branch_name>.md`.** The branch name is `scapy_0925ada48540`, so the document is `scapy_0925ada48540.md`.
- **Place the document in `blitzy/documentation/`.** The output directory is `blitzy/documentation/` in the destination repository.
- **Temporary scripts must be cleaned up.** Any observation scripts used during analysis must be removed after use, leaving no artifacts in the repository.
- **Include Mermaid diagrams for complex control flows.** The dissection pipeline, binding decision logic, and tunnel recursion should be visualized.
- **Use source code citations for all technical details.** Every technical claim must reference the specific file, class, method, and line number in the Scapy codebase.
- **Document the HTTP layer loading caveat.** Explicitly note that HTTP is not in the default `load_layers` list and requires `load_layer('http')` for port-80 traffic to be recognized.
- **Show runtime evidence for each question.** Each answer must include actual Scapy runtime output (`show()`, `repr()`, or similar) demonstrating the described behavior, not just code analysis.


## 0.11 References

### 0.11.1 Files and Folders Searched

The following files and folders were directly inspected during context gathering to derive the conclusions in this Agent Action Plan:

**Core dissection engine files:**

| File | Lines Examined | Key Findings |
|---|---|---|
| `scapy/packet.py` | 77–200, 340–420, 982–1100, 1696–1930, 1920–2060 | `Packet` class, `dissect()`, `do_dissect()`, `do_dissect_payload()`, `guess_payload_class()`, `default_payload_class()`, `add_payload()`, `extract_padding()`, `Raw`, `Padding`, `NoPayload`, `bind_layers()`, `bind_bottom_up()`, `bind_top_down()` |
| `scapy/base_classes.py` | 281–375 | `Packet_metaclass.__new__()`, `aliastypes` construction, `fields_desc` resolution |
| `scapy/config.py` | 847–900 | `load_layers` list (48 default modules), HTTP/CAN/TLS not loaded by default |
| `scapy/fields.py` | (summary only) | 60+ field types with `getfield()` / `addfield()` API |

**Protocol layer files:**

| File | Lines Examined | Key Findings |
|---|---|---|
| `scapy/layers/l2.py` | 244–310, 568–716 | `Ether` (3 fields: dst, src, type), `dispatch_hook()` Ether/Dot3 selection, `GRE` class (proto field, conditional fields), all `bind_layers()` registrations (Ether→LLC, Ether→Dot1Q, Ether→ARP, GRE→LLC, GRE→Ether, GRE→ARP, GRE→GRErouting) |
| `scapy/layers/inet.py` | 521–600, 1095–1130 | `IP` class (version, ihl, proto, etc.), `IP.extract_padding()`, `TCP` class, binding registrations: `Ether→IP (type=2048)`, `GRE→IP (proto=2048)`, `IP→TCP (proto=6)`, `IP→UDP (proto=17)`, `IP→GRE (proto=47)`, `IP→ICMP (proto=1)` |
| `scapy/layers/inet6.py` | 4083–4100 | `Ether→IPv6 (type=0x86dd)`, `GRE→IPv6 (proto=0x86dd)`, `IPv6→TCP (nh=6)`, `IPv6→GRE (nh=47)` |
| `scapy/layers/http.py` | 425–460, 548–660, 740–754 | `HTTP` class, `HTTP.guess_payload_class()` with regex matching, `bind_bottom_up(TCP, HTTP, sport=80)`, `bind_bottom_up(TCP, HTTP, dport=80)`, `bind_layers(TCP, HTTP, sport=80, dport=80)` |

**Documentation files:**

| File | Lines Examined | Key Findings |
|---|---|---|
| `doc/scapy/build_dissect.rst` | 1–500 | Existing conceptual coverage of dissection lifecycle, `do_dissect()`, `guess_payload_class()`, `bind_layers()`, `payload_guess` — provides prior art but lacks runtime evidence |
| `doc/scapy/usage.rst` | 1–50 | Interactive shell usage, dissection performance tips, session filtering |
| `doc/scapy/conf.py` | 1–120 | Sphinx configuration: `>=3.0.0`, `sphinx_rtd_theme`, `autodoc`, `napoleon`, custom `scapy_doc` extension |
| `doc/scapy/index.rst` | Full | Toctree structure for Sphinx manual |
| `.readthedocs.yml` | Full | RTD config: Ubuntu 20.04, Python 3.9, EPUB/PDF, `pip install .[docs]` |

**Configuration and project files:**

| File | Lines Examined | Key Findings |
|---|---|---|
| `pyproject.toml` | Full | Python `>=3.7, <4`, setuptools build, optional `[docs]` extra with Sphinx deps |
| `tox.ini` | 1–50 | Test matrix: py27–py311, multi-platform, docs environment |
| `README.md` | 1–40 | Project overview, supported platforms, quick-start guide |

**Folders explored:**

| Folder | Depth | Key Findings |
|---|---|---|
| `` (root) | Level 0 | 5 children: `scapy/`, `.config/`, `.github/`, `doc/`, `test/` |
| `scapy/` | Level 1 | 34 files + 7 subfolders; core runtime package |
| `scapy/layers/` | Level 2 | 48+ protocol layer modules + `tls/` subfolder |
| `doc/` | Level 1 | 4 children: `notebooks/`, `scapy/`, `syntax/`, `vagrant_ci/` |
| `doc/scapy/` | Level 2 | Sphinx source: RST chapters, conf.py, Makefile, layers/ |

### 0.11.2 Attachments

No attachments were provided by the user. No Figma screens, design files, or supplementary documents were supplied.

### 0.11.3 Runtime Validation Performed

The following runtime validations were executed during context gathering to verify code-level conclusions:

- **Import validation:** Confirmed `scapy.layers.l2.Ether`, `scapy.layers.inet.IP`, `scapy.layers.inet.TCP`, `scapy.layers.l2.GRE`, `scapy.layers.http.HTTP` are all importable
- **Binding table enumeration:** Programmatically listed `payload_guess` for `Ether` (16 entries), `IP` (11 entries), `GRE` (9 entries), and `TCP` (21+ entries) to verify binding registrations
- **HTTP binding verification:** Confirmed HTTP bindings exist for TCP sport=80, dport=80, sport=8080, dport=8080, and the combined sport=80+dport=80
- **HTTP not in default layers:** Verified that `http` is not in `scapy/config.py`'s `load_layers` list (per line 847 comment: *"can, tls, http and a few others are not loaded by default"*)


