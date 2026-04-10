# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

### 0.1.1 Core Documentation Objective

Based on the provided requirements, the Blitzy platform understands that the documentation objective is to **create new documentation** that comprehensively answers a series of deeply interrelated questions about DNS name compression in the Scapy packet manipulation library. The user is onboarding into the Scapy codebase and seeks a thorough, code-grounded explanation of how the DNS name compression subsystem operates during both packet parsing (decompression) and packet building (compression).

**Documentation Type:** Technical Q&A / Deep-dive architecture explainer

**Request Category:** Create new documentation

The user's questions encompass the following specific topics, each requiring detailed analysis of the source code at `scapy/layers/dns.py`:

- **Wire-format encoding of domain names** — How domain names are encoded into length-prefixed label sequences before compression enters the picture, via the `dns_encode()` function (line 154).
- **Compression pointer detection** — How the `0xc0` bitmask distinguishes a compression pointer from a normal label length byte, via the `cur & 0xc0` check in `dns_get_str()` (line 103).
- **Decompression walkthrough** — How `dns_get_str()` (line 69) unravels a chain of compression pointer references into a readable domain name, and where the unwinding begins within the DNS dissection pipeline.
- **Loop detection and prevention** — How the `processed_pointers` list in `dns_get_str()` (line 88) prevents the parser from chasing circular pointer references, and what happens when a loop is caught (a warning is emitted and parsing breaks).
- **Error handling for malformed packets** — What occurs when compression pointers reference out-of-bounds locations, truncated data, or invalid offsets, including the `log_runtime.info` calls at lines 97–99 and 109–111.
- **Cross-boundary reference resolution** — How the `InheritOriginDNSStrPacket` class (line 270) and its `_orig_s` attribute provide access to the full original packet bytes for resolving pointers that span across DNS record boundaries.
- **Compression during packet building** — How the `dns_compress()` function (line 184) decides which name parts to compress, how it discovers compression opportunities by walking through all DNS string fields, and the strategy behind `possible_shortens()` and `field_gen()`.
- **Consistency of decompression** — Whether different compression pointer strategies for the same domain always decompress to identical strings.

### 0.1.2 Special Instructions and Constraints

The user has specified the following critical directives:

- **Repository must remain unchanged:** "Temporary scripts may be used for observation, but the repository itself should remain unchanged and anything temporary should be cleaned up afterward." This means no existing source files in the Scapy repository may be modified.
- **Implementation rule (SWE-AtlasQnA-Repo):** The output must be a new markdown document named `scapy_0925ada48540.md`, placed in the `blitzy/documentation/` directory. The document must provide thinking and rationale behind answers, base all answers on the code as truth, and not make assumptions.
- **Code-as-truth principle:** All explanations must be grounded in the actual implementation at `scapy/layers/dns.py`, with source citations referencing specific line numbers and function names.
- **No assumptions:** Answers must be derived directly from reading and analyzing the source code, not from external assumptions about how DNS compression "should" work.

### 0.1.3 Technical Interpretation

These documentation requirements translate to the following technical documentation strategy:

- To document DNS wire-format encoding, we will analyze and explain the `dns_encode()` function at `scapy/layers/dns.py:154–172`, detailing how it splits a dotted domain name on `"."` delimiters, prepends each label with its length byte, truncates labels exceeding 63 bytes, and terminates with a `\x00` null byte.
- To document compression pointer detection, we will trace the bit-masking logic at `scapy/layers/dns.py:103` where `cur & 0xc0` identifies the two highest bits being set, distinguishing a pointer from a normal label length (which is limited to 0–63).
- To document the full decompression flow, we will provide a step-by-step walkthrough of `dns_get_str()` at `scapy/layers/dns.py:69–144`, covering initialization, the main `while True` loop, pointer following via the formula at line 114, label accumulation at line 134, and return value construction at line 144.
- To document loop protection, we will explain the `processed_pointers` list initialized at line 88 and the membership check at line 115 that triggers a `warning("DNS decompression loop detected")` at line 116 and breaks the loop.
- To document error handling, we will catalog all error paths in `dns_get_str()`: premature end detection at lines 96–100, incomplete jump token handling at lines 108–112, and the `Scapy_Exception` raised at lines 128–129 when the full packet is unavailable for decompression.
- To document cross-boundary resolution, we will explain `InheritOriginDNSStrPacket` (line 270), how `_orig_s` is passed during record decoding at `DNSRRField.decodeRR()` (line 360) and `DNSQRField.decodeRR()` (line 403), and how `dns_get_str()` switches from the local byte string `s` to the full packet `s_full` at lines 120–125.
- To document compression during building, we will analyze `dns_compress()` (lines 184–267), its inner generators `field_gen()` (line 194) and `possible_shortens()` (line 211), the pointer encoding at lines 227–229, and the replacement strategy at lines 242–257.
- To document decompression consistency, we will reason from the deterministic pointer-following logic in `dns_get_str()` to show that any chain of valid pointers resolving to the same byte sequence will produce identical output strings.

### 0.1.4 Inferred Documentation Needs

Based on code analysis:

- The `dns_get_str()` function's return tuple contains four elements `(name, pointer, bytes_left, was_compressed)` as seen at line 144, but this is not documented in any existing file. The new documentation should explain this interface.
- The `-12` offset in the pointer calculation at line 114 (`pointer = ((cur & ~0xc0) << 8) + orb(s[pointer]) - 12`) reflects that Scapy's internal DNS byte buffer starts after the 12-byte DNS header, so pointer values from the wire (which are offsets from the start of the DNS message) must be adjusted. This nuance is critical and must be clearly explained.
- The `_is_ptr()` helper at line 147 provides a check for already-built/compressed DNS strings, which is used by `dns_encode()` to avoid double-encoding. This interplay should be documented for completeness.
- The `DNSStrField.getfield()` method at line 300 is the bridge between the field system and the decompression engine. Its role in invoking `dns_get_str()` during dissection must be documented.
- The `dns_compress()` function's approach of encoding names, finding their byte offset via `build_pkt.index(encoded)`, and then replacing subsequent occurrences represents a specific greedy first-occurrence strategy that should be explained.


## 0.2 Documentation Discovery and Analysis

### 0.2.1 Existing Documentation Infrastructure Assessment

Repository analysis reveals a mature Sphinx-based documentation ecosystem with no dedicated DNS layer documentation page.

**Documentation Framework:** Sphinx `>=3.0.0` with the `sphinx_rtd_theme>=0.4.3` theme, as declared in `pyproject.toml` under `[project.optional-dependencies] docs`. The documentation source lives under `doc/scapy/` and is hosted on ReadTheDocs at `https://scapy.readthedocs.io`.

**Documentation Generator Configuration:** `doc/scapy/conf.py` configures Sphinx with the extensions `sphinx.ext.autodoc`, `sphinx.ext.napoleon`, `sphinx.ext.todo`, `sphinx.ext.linkcode`, and a custom `scapy_doc` extension. The `needs_sphinx` minimum version is `3.0.0`.

**API Documentation Tools:** Sphinx `autodoc` generates API documentation from Python docstrings. The `apitree` tox environment runs `sphinx-apidoc` to regenerate the API tree. The `dns_get_str()`, `dns_encode()`, and `dns_compress()` functions have docstrings, but they are minimal and focus on parameter descriptions rather than behavioral explanation.

**Diagram Tools Detected:** No Mermaid or PlantUML configurations were found in the documentation infrastructure. The existing docs use ReStructuredText with code blocks and inline examples. The new document will use Mermaid diagrams within the markdown file as permitted by the output format.

**Documentation Hosting/Deployment:** ReadTheDocs is configured via `.readthedocs.yml` with Ubuntu 20.04, Python 3.9, and `pip install .[docs]`. The `tox.ini` defines a `[testenv:docs]` environment that runs `sphinx-build -W --keep-going -b html . _build/html`.

**Existing Documentation Files Examined:**

| Path | Content | DNS Coverage |
|------|---------|--------------|
| `doc/scapy/usage.rst` | General usage guide with protocol examples | Brief DNS query examples (lines 1356–1377); no compression coverage |
| `doc/scapy/build_dissect.rst` | Field system and dissector construction guide | Lists `DNSStrField`, `DNSRRCountField`, `DNSRRField`, `DNSQRField` as field types (line 1091) with no behavioral documentation |
| `doc/scapy/layers/index.rst` | Layer documentation index | Glob pattern for layer-specific docs; no `dns.rst` file exists |
| `doc/scapy/advanced_usage.rst` | Advanced techniques | No DNS compression content |
| `doc/scapy/extending.rst` | Extension guide | No DNS-specific content |
| `README.md` | Project overview | No DNS internals documentation |

### 0.2.2 Repository Code Analysis for Documentation

Search patterns used to locate all code relevant to DNS name compression:

- **Primary DNS module:** `scapy/layers/dns.py` (1179 lines) — Contains all compression/decompression logic, DNS packet classes, field types, and utility functions.
- **Test suite:** `test/scapy/layers/dns.uts` (241 lines) — Contains regression tests for compression, decompression, loop detection, premature end handling, and round-trip fidelity.
- **DNSSEC tests:** `test/scapy/layers/dns_dnssec.uts` (146 lines) — DNSSEC-specific tests that exercise `DNSStrField` compression on RRSIG/NSEC records.
- **EDNS0 tests:** `test/scapy/layers/dns_edns0.uts` (91 lines) — EDNS0-specific tests.
- **Compatibility module:** `scapy/compat.py` — Defines `orb()` (line 146) and `chb()` (line 140) used throughout DNS code for byte-level operations.
- **Error module:** `scapy/error.py` — Defines `log_runtime` (line 123), `warning()` (line 132), and `Scapy_Exception` (line 30).
- **Packet base:** `scapy/packet.py` — Defines the `Packet` base class from which `InheritOriginDNSStrPacket` inherits.
- **Fields base:** `scapy/fields.py` — Defines `StrLenField` from which `DNSStrField` inherits.
- **Cross-module usage:** `DNSStrField` is reused by `scapy/layers/dcerpc.py` (line 264), `scapy/layers/dhcp6.py` (line 815), `scapy/contrib/socks.py` (line 132), and inspired implementations in `scapy/contrib/pfcp.py` (line 414) and `scapy/contrib/gtp.py` (line 589).
- **LLMNR module:** `scapy/layers/llmnr.py` imports `DNSQRField`, `DNSRRField`, `DNSRRCountField`, and `DNS_am` from the DNS module, confirming the DNS compression infrastructure is shared by LLMNR.

**Key Directories Examined:**

| Directory | Relevance |
|-----------|-----------|
| `scapy/layers/` | Contains `dns.py` and related protocol modules |
| `test/scapy/layers/` | Contains all DNS test files (`dns.uts`, `dns_dnssec.uts`, `dns_edns0.uts`) |
| `doc/scapy/` | Documentation root with Sphinx configuration |
| `doc/scapy/layers/` | Layer-specific documentation (no `dns.rst` exists) |
| `blitzy/documentation/` | Target directory for new documentation (does not yet exist) |

### 0.2.3 Web Search Research Conducted

No web searches were required for this documentation task. The user explicitly stated that answers must be based on the code as truth, and all DNS compression questions can be fully answered from direct source code analysis of `scapy/layers/dns.py` and its test suite. The DNS wire format (RFC 1035 Section 4.1.4) is a well-established standard, and Scapy's implementation is self-contained in the functions analyzed above.


## 0.3 Documentation Scope Analysis

### 0.3.1 Code-to-Documentation Mapping

The user's questions map entirely to the DNS name compression subsystem within `scapy/layers/dns.py`. Every question can be answered by analyzing specific functions, classes, and methods in this single file, with supporting context from the compatibility and error modules.

**Module: `scapy/layers/dns.py`**

- Public APIs requiring documentation:
  - `dns_get_str(s, pointer=0, pkt=None, _fullpacket=False)` — line 69, the core decompression function
  - `dns_encode(x, check_built=False)` — line 154, the name-to-wire-format encoder
  - `dns_compress(pkt)` — line 184, the packet-level compression function
  - `_is_ptr(x)` — line 147, helper to detect already-compressed names
  - `DNSgetstr(*args, **kwargs)` — line 175, deprecated legacy wrapper
- Classes requiring documentation:
  - `InheritOriginDNSStrPacket` — line 270, base class that carries `_orig_s` for cross-boundary decompression
  - `DNSStrField` — line 279, the field type that bridges Scapy's field system to the compression engine
  - `DNS` — line 462, the main DNS packet class with `compress()` method at line 514
  - `DNSRRField` — line 336, the resource record field that orchestrates per-record decompression
  - `DNSQRField` — line 399, the question record field
  - `DNSRR` — line 1034, the standard resource record class
  - `DNSQR` — line 454, the question record class
- Current documentation status: **Minimal inline docstrings only.** `dns_get_str()` has a 7-line docstring. `dns_encode()` has a 5-line docstring. `dns_compress()` has a 1-line docstring. No prose documentation exists.
- Documentation needed: Complete behavioral explanation, step-by-step walkthrough, error path catalog, and cross-reference documentation.

**Module: `scapy/compat.py` (supporting)**

- Functions referenced: `orb()` (line 146) — converts byte to int; `chb()` (line 140) — converts int to single byte. These are used throughout the DNS code and need brief contextual explanation.
- Current documentation status: Has inline docstrings.
- Documentation needed: Brief mention in context of DNS code explanation.

**Module: `scapy/error.py` (supporting)**

- Functions referenced: `log_runtime` logger (line 123), `warning()` (line 132), `Scapy_Exception` (line 30).
- Documentation needed: Brief explanation of how DNS error paths use these mechanisms.

### 0.3.2 Documentation Gap Analysis

Given the requirements and repository analysis, documentation gaps include:

- **No existing documentation for DNS name compression internals.** The `doc/scapy/layers/` directory has no `dns.rst` file. The only DNS mentions are brief usage examples in `doc/scapy/usage.rst` showing simple query construction.
- **No architectural explanation of the decompression pipeline.** The flow from raw bytes → `DNS.__init__()` → `DNSRRField.getfield()` → `dns_get_str()` → fully resolved domain name is not documented anywhere.
- **No documentation of error handling behavior.** The specific responses to malformed compression pointers, loops, truncated data, and out-of-bounds references are not described in any user-facing documentation.
- **No documentation of the `InheritOriginDNSStrPacket` mechanism.** The `_orig_s` attribute and its role in enabling cross-record-boundary pointer resolution is not explained in any existing documentation, despite being architecturally significant.
- **No documentation of the compression algorithm in `dns_compress()`.** The greedy first-occurrence strategy, the `field_gen()` iterator, the `possible_shortens()` suffix generator, and the pointer encoding formula are undocumented beyond a single-line docstring.
- **No documentation of the `-12` offset adjustment.** The critical detail that Scapy's internal DNS buffer does not include the 12-byte DNS header, requiring pointer values to be adjusted by subtracting 12, is not explained anywhere.

All of these gaps are addressed by the new markdown document to be created at `blitzy/documentation/scapy_0925ada48540.md`.


## 0.4 Documentation Implementation Design

### 0.4.1 Documentation Structure Planning

The output is a single comprehensive markdown document that answers all of the user's questions in a logical, progressive sequence. Per the SWE-AtlasQnA-Repo implementation rule, the file is placed at:

```
blitzy/
└── documentation/
    └── scapy_0925ada48540.md
```

The document's internal structure follows a progression from foundational concepts to advanced behavior:

- **Section 1: Wire-Format Encoding** — How domain names are encoded into length-prefixed label sequences (`dns_encode()`)
- **Section 2: Compression Pointer Detection** — How the `0xc0` bitmask distinguishes pointers from label lengths
- **Section 3: Decompression Walkthrough** — Step-by-step trace through `dns_get_str()` showing the full unwinding process
- **Section 4: Where Decompression Begins** — How the DNS dissection pipeline connects to `dns_get_str()` through `DNSStrField`, `DNSRRField`, and `DNSQRField`
- **Section 5: Loop Detection and Prevention** — The `processed_pointers` mechanism and what happens when a loop is caught
- **Section 6: Error Handling for Malformed Packets** — Out-of-bounds pointers, truncated data, incomplete jump tokens
- **Section 7: Cross-Boundary Reference Resolution** — The `InheritOriginDNSStrPacket` mechanism and `_orig_s`
- **Section 8: Compression During Packet Building** — How `dns_compress()` finds and applies compression opportunities
- **Section 9: Decompression Consistency** — Whether different compression strategies produce identical results
- **Section 10: End-to-End Flow Summary** — From raw bytes to resolved names and back

### 0.4.2 Content Generation Strategy

**Information Extraction Approach:**

- Extract function signatures and behavioral logic from `scapy/layers/dns.py` using direct code reading at specific line ranges
- Generate explanatory examples by analyzing the test assertions in `test/scapy/layers/dns.uts`, particularly the decompression tests at lines 55–80, the compression tests at lines 103–128, the loop detection test at line 150, and the premature end tests at lines 155–159
- Create flow diagrams by mapping the control flow within `dns_get_str()` and `dns_compress()`
- Provide inline code citations referencing `scapy/layers/dns.py:<line>` for every technical claim

**Documentation Standards:**

- Markdown formatting with proper headers (`#`, `##`, `###`)
- Mermaid diagram integration using fenced code blocks for flow visualization
- Code examples using fenced Python blocks with syntax highlighting
- Source citations as inline references in the format `Source: scapy/layers/dns.py:LineNumber`
- Tables for parameter descriptions, error catalogs, and function signatures
- Thinking/rationale provided alongside each answer as required by the SWE-AtlasQnA-Repo rule

### 0.4.3 Diagram and Visual Strategy

Mermaid diagrams to create:

- **Decompression flowchart** — A flowchart tracing the `while True` loop in `dns_get_str()`, showing the three code paths (label, pointer, terminator) and the error/loop-detection branches
- **DNS dissection pipeline** — A sequence diagram showing how raw bytes flow from `DNS.__init__()` through `DNSRRField.getfield()` and `DNSQRField.decodeRR()` into `dns_get_str()`
- **Compression algorithm flowchart** — A flowchart showing how `dns_compress()` iterates through fields, generates possible shortenings, and applies pointer replacements
- **Wire-format encoding diagram** — A visual representation of how `www.example.com` becomes `\x03www\x07example\x03com\x00`


## 0.5 Documentation File Transformation Mapping

### 0.5.1 File-by-File Documentation Plan

| Target Documentation File | Transformation | Source Code/Docs | Content/Changes |
|---------------------------|----------------|------------------|-----------------|
| `blitzy/documentation/scapy_0925ada48540.md` | CREATE | `scapy/layers/dns.py`, `scapy/compat.py`, `scapy/error.py`, `scapy/packet.py`, `test/scapy/layers/dns.uts` | Comprehensive Q&A document answering all user questions about DNS name compression in Scapy, with code-grounded explanations, Mermaid diagrams, rationale, and source citations |

**Detailed transformation notes:**

- Only one file is created. No existing files are modified, deleted, or used as templates.
- The document is self-contained and does not require integration into the Sphinx documentation tree or ReadTheDocs pipeline.
- The `blitzy/documentation/` directory does not yet exist and must be created.

### 0.5.2 New Documentation File Detail

```
File: blitzy/documentation/scapy_0925ada48540.md
Type: Technical Q&A / Deep-dive explainer
Source Code Files:
    - scapy/layers/dns.py (primary — all compression/decompression logic)
    - scapy/compat.py (supporting — orb(), chb() byte helpers)
    - scapy/error.py (supporting — log_runtime, warning(), Scapy_Exception)
    - scapy/packet.py (supporting — Packet base class, dissection pipeline)
    - scapy/fields.py (supporting — StrLenField base for DNSStrField)
    - test/scapy/layers/dns.uts (supporting — test cases as behavioral evidence)
Sections:
    - DNS Wire-Format Encoding (dns_encode at line 154)
    - Compression Pointer Detection (0xc0 bitmask at line 103)
    - Decompression Walkthrough (dns_get_str at line 69)
    - Where Decompression Begins in the Pipeline (DNSStrField.getfield at line 300, DNSRRField.getfield at line 375, DNSQRField.decodeRR at line 400)
    - Loop Detection and Prevention (processed_pointers at line 88, check at line 115)
    - Error Handling for Malformed Packets (premature end at line 96, incomplete jump at line 108, Scapy_Exception at line 128)
    - Cross-Boundary Reference Resolution (InheritOriginDNSStrPacket at line 270, _orig_s mechanism)
    - Compression During Packet Building (dns_compress at line 184, field_gen at line 194, possible_shortens at line 211)
    - Decompression Consistency Analysis
    - End-to-End Flow Summary
Diagrams:
    - Mermaid flowchart for dns_get_str() decompression loop
    - Mermaid sequence diagram for DNS dissection pipeline
    - Mermaid flowchart for dns_compress() algorithm
Key Citations:
    - scapy/layers/dns.py:69-144 (dns_get_str)
    - scapy/layers/dns.py:147-151 (_is_ptr)
    - scapy/layers/dns.py:154-172 (dns_encode)
    - scapy/layers/dns.py:184-267 (dns_compress)
    - scapy/layers/dns.py:270-276 (InheritOriginDNSStrPacket)
    - scapy/layers/dns.py:279-307 (DNSStrField)
    - scapy/layers/dns.py:336-396 (DNSRRField)
    - scapy/layers/dns.py:399-405 (DNSQRField)
    - scapy/layers/dns.py:454-459 (DNSQR)
    - scapy/layers/dns.py:462-516 (DNS)
    - scapy/layers/dns.py:1034-1061 (DNSRR)
    - test/scapy/layers/dns.uts:55-80 (decompression tests)
    - test/scapy/layers/dns.uts:103-128 (compression tests)
    - test/scapy/layers/dns.uts:141-159 (dns_get_str edge cases)
```

### 0.5.3 Documentation Configuration Updates

No documentation configuration files require modification. The new document is a standalone markdown file in the `blitzy/documentation/` directory and is not integrated into the Sphinx build pipeline or ReadTheDocs deployment. This aligns with the implementation rule that prohibits modifying existing repository files.

### 0.5.4 Cross-Documentation Dependencies

- **No shared content or includes:** The document is entirely self-contained.
- **No navigation links to update:** The document is not part of the existing Sphinx documentation tree.
- **No table of contents updates required:** The `doc/scapy/index.rst` and `doc/scapy/layers/index.rst` files are not modified.
- **No index or glossary updates needed:** The document defines its own terms inline.


## 0.6 Dependency Inventory

### 0.6.1 Documentation Dependencies

This documentation task requires no additional tools or packages beyond the Python runtime needed to read and analyze the source code. The output is a static markdown file with no build step.

| Registry | Package Name | Version | Purpose |
|----------|--------------|---------|---------|
| PyPI | scapy | 2.7.0 (from `scapy/VERSION` / `pyproject.toml`) | Subject of documentation; source code analyzed directly |
| PyPI | sphinx | >=3.0.0 | Existing documentation framework (not used for this task) |
| PyPI | sphinx_rtd_theme | >=0.4.3 | Existing documentation theme (not used for this task) |
| System | Python | >=3.7, <4 (highest documented: 3.10 per classifiers) | Runtime for any temporary observation scripts |

No new dependencies need to be installed for this documentation task. The markdown file is produced by analyzing the source code directly and requires no generation tools, diagram renderers, or documentation builders.

### 0.6.2 Documentation Reference Updates

Not applicable. No existing documentation files require link updates because:

- The new file `blitzy/documentation/scapy_0925ada48540.md` is not referenced by any existing documentation.
- No existing documentation files are being modified.
- The SWE-AtlasQnA-Repo rule explicitly states: "Do not modify any existing files in the source repository."


## 0.7 Coverage and Quality Targets

### 0.7.1 Documentation Coverage Metrics

**Current coverage of DNS compression internals: 0%.** No existing documentation in the repository explains the decompression algorithm, compression strategy, error handling, or cross-boundary resolution mechanisms.

**Target coverage:** 100% of the user's questions, mapped to the following specific functions and code paths:

| User Question Topic | Code Target | Current Coverage | Target Coverage |
|---------------------|-------------|------------------|-----------------|
| Wire-format encoding of domain names | `dns_encode()` (line 154) | 5-line docstring only | Full behavioral explanation with examples |
| How `0xc0` distinguishes pointers from labels | `dns_get_str()` line 103 | No documentation | Complete bitmask explanation with rationale |
| How decompression unravels pointer chains | `dns_get_str()` lines 69–144 | 7-line docstring only | Step-by-step walkthrough with Mermaid diagram |
| Where unwinding begins in the pipeline | `DNSStrField.getfield()` line 300, `DNSRRField.getfield()` line 375 | No documentation | Full pipeline trace from `DNS.__init__()` to resolved name |
| Loop detection and prevention | `processed_pointers` at line 88, check at line 115 | No documentation | Complete mechanism explanation with behavior on detection |
| Out-of-bounds / truncated pointer handling | Lines 96–100, 108–112, 128–129 | No documentation | Full error path catalog with outcomes |
| Cross-boundary reference resolution | `InheritOriginDNSStrPacket` line 270, `_orig_s` | No documentation | Architectural explanation of the `_orig_s` mechanism |
| Compression strategy during building | `dns_compress()` lines 184–267 | 1-line docstring only | Complete algorithm walkthrough with Mermaid diagram |
| Consistency of decompression results | `dns_get_str()` determinism | No documentation | Reasoning from code logic with supporting test evidence |

### 0.7.2 Documentation Quality Criteria

**Completeness requirements:**
- Every user question is answered with a dedicated section
- Every answer cites specific source code locations (file:line)
- Every answer includes rationale/thinking as required by the SWE-AtlasQnA-Repo rule
- All error paths and edge cases mentioned by the user are fully documented

**Accuracy validation:**
- All code references verified against `scapy/layers/dns.py` at the current commit (`0925ada4`)
- All behavioral claims validated against test assertions in `test/scapy/layers/dns.uts`
- No assumptions made beyond what the code explicitly demonstrates

**Clarity standards:**
- Progressive disclosure: wire-format basics → compression detection → decompression walkthrough → error handling → building → consistency
- Consistent use of code citation format: `Source: scapy/layers/dns.py:<line>`
- Technical accuracy with accessible language for an onboarding developer

**Maintainability:**
- Source citations enable future readers to verify claims against the codebase
- Self-contained document with no external dependencies

### 0.7.3 Example and Diagram Requirements

- **Minimum examples per topic:** At least one concrete code or byte-level example per question answered
- **Diagram types required:** Mermaid flowcharts for `dns_get_str()` and `dns_compress()` control flow; Mermaid sequence diagram for the dissection pipeline
- **Code example testing:** Examples are derived from existing test assertions in `test/scapy/layers/dns.uts` which are part of the CI-validated test suite
- **Visual content freshness:** Diagrams reflect the code at commit `0925ada4` (current HEAD of the working branch)


## 0.8 Scope Boundaries

### 0.8.1 Exhaustively In Scope

**New documentation files:**
- `blitzy/documentation/scapy_0925ada48540.md` — The single output artifact of this task

**Source code analyzed for documentation content:**
- `scapy/layers/dns.py` — All functions, classes, and methods related to DNS name compression and decompression (lines 69–307, 336–405, 454–516, 1034–1061)
- `scapy/compat.py` — `orb()` and `chb()` helpers (lines 140–152)
- `scapy/error.py` — `log_runtime`, `warning()`, `Scapy_Exception` (lines 30, 123–137)
- `scapy/packet.py` — `Packet` base class (for inheritance context of `InheritOriginDNSStrPacket`)
- `scapy/fields.py` — `StrLenField` base class (for inheritance context of `DNSStrField`)
- `test/scapy/layers/dns.uts` — All test cases related to compression, decompression, loop detection, and edge cases (lines 55–241)

**Documentation topics covered:**
- DNS wire-format encoding (`dns_encode()`)
- Compression pointer detection (`0xc0` bitmask)
- Decompression algorithm (`dns_get_str()`)
- DNS dissection pipeline integration (`DNSStrField`, `DNSRRField`, `DNSQRField`)
- Loop detection and prevention (`processed_pointers`)
- Error handling for malformed packets (premature end, incomplete jump, out-of-bounds, `Scapy_Exception`)
- Cross-boundary reference resolution (`InheritOriginDNSStrPacket`, `_orig_s`)
- Compression during packet building (`dns_compress()`, `field_gen()`, `possible_shortens()`)
- Decompression consistency under varying compression strategies
- End-to-end flow from raw bytes to resolved domain names

### 0.8.2 Explicitly Out of Scope

- **Source code modifications** — No files in the Scapy repository are modified. The SWE-AtlasQnA-Repo rule states: "Do not modify any existing files in the source repository."
- **Test file modifications** — No test files are created or modified.
- **Sphinx documentation updates** — The new document is not integrated into `doc/scapy/` or the Sphinx build pipeline.
- **DNSSEC-specific documentation** — While DNSRR subclasses like `DNSRRRSIG`, `DNSRRNSEC`, and `DNSRRDNSKEY` use `DNSStrField`, detailed DNSSEC documentation is out of scope unless directly relevant to compression behavior.
- **EDNS0-specific documentation** — `DNSRROPT` and EDNS0 extensions are out of scope unless directly relevant to compression.
- **DNS answering machine (`DNS_am`)** — The DNS spoofing/responding machinery at line 1114 is out of scope as it does not relate to compression.
- **Dynamic DNS functions (`dyndns_add`, `dyndns_del`)** — Lines 1074–1111 are out of scope.
- **Non-DNS protocol layers** — All other protocol modules in `scapy/layers/` are out of scope.
- **Deployment or CI/CD changes** — No changes to `.readthedocs.yml`, `tox.ini`, `.github/workflows/`, or any other infrastructure files.
- **Feature additions or code refactoring** — No code changes of any kind.
- **Documentation for `DNSStrField` usage in external modules** — While `dcerpc.py`, `dhcp6.py`, and `socks.py` use `DNSStrField`, documenting their usage is out of scope.


## 0.9 Execution Parameters

### 0.9.1 Documentation-Specific Instructions

- **Documentation build command:** Not applicable — output is a standalone markdown file, not part of a documentation build system.
- **Documentation preview command:** The markdown file can be previewed with any markdown renderer. For repository-context preview: `cat blitzy/documentation/scapy_0925ada48540.md`.
- **Diagram generation command:** Not applicable — Mermaid diagrams are embedded inline in the markdown and rendered by any Mermaid-capable viewer (GitHub, VS Code with Mermaid extension, etc.).
- **Documentation deployment command:** Not applicable — standalone file.
- **Default format:** Markdown with Mermaid diagrams.
- **Citation requirement:** Every technical claim must reference the specific source file and line number where the behavior is implemented.
- **Style guide:** Follow the SWE-AtlasQnA-Repo rules: provide thinking/rationale behind answers; base all answers on the code as truth; do not make assumptions.
- **Documentation validation:** Manual review against the nine user questions to confirm each is comprehensively answered with code citations.
- **Temporary scripts policy:** If any temporary Python scripts are created for observation during documentation authoring, they must be cleaned up afterward and must not modify the repository.

### 0.9.2 Output File Naming Convention

Per the SWE-AtlasQnA-Repo implementation rule: "Create a new markdown document named `<source_branch_name>.md`." The current branch name is `scapy_0925ada48540`, therefore the output file is:

```
blitzy/documentation/scapy_0925ada48540.md
```


## 0.10 Rules for Documentation

The following rules are derived from the user's explicit instructions and the SWE-AtlasQnA-Repo implementation rule:

- **"Do not modify any existing files in the source repository."** — No files in the Scapy repository may be edited, deleted, or renamed. Only the new file `blitzy/documentation/scapy_0925ada48540.md` is created.
- **"Provide thinking / rationale behind the answers."** — Each answer in the documentation must include reasoning that explains *why* the code works as described, not just *what* it does.
- **"Do not make assumptions, base your answers on the code as the truth."** — All behavioral claims must be traceable to specific lines in the source code. No external assumptions about DNS compression behavior are permitted.
- **"Place the generated document in the `blitzy/documentation` directory in the destination repo."** — The output directory must be `blitzy/documentation/` relative to the repository root.
- **"Create a new markdown document named `<source_branch_name>.md`."** — The file must be named `scapy_0925ada48540.md` matching the branch name exactly.
- **"Temporary scripts may be used for observation, but the repository itself should remain unchanged and anything temporary should be cleaned up afterward."** — Any temporary observation scripts must be deleted after use.
- **"Comprehensively answers the question(s) posed in the prompt."** — Every question raised by the user must receive a thorough, complete answer. No question may be deferred or left partially answered.


## 0.11 References

### 0.11.1 Files and Folders Searched

The following files and folders were directly examined during context gathering for this Agent Action Plan:

**Primary Source Files Read:**

| File Path | Lines Examined | Purpose |
|-----------|---------------|---------|
| `scapy/layers/dns.py` | 1–1179 (full file) | Core DNS module containing all compression/decompression logic, field types, packet classes, and utility functions |
| `test/scapy/layers/dns.uts` | 1–241 (full file) | DNS test suite with regression tests for compression, decompression, loop detection, error handling, and round-trip fidelity |
| `scapy/compat.py` | 1–160 | Compatibility module defining `orb()` and `chb()` byte-manipulation helpers used in DNS code |
| `scapy/error.py` | 1–137 | Error module defining `log_runtime`, `warning()`, and `Scapy_Exception` used in DNS error paths |
| `pyproject.toml` | Full file | Project metadata, Python version requirements, dependency declarations, documentation extras |
| `tox.ini` | docs section | Documentation build configuration (`sphinx-build` command, deps) |
| `.readthedocs.yml` | Full file | ReadTheDocs hosting configuration (Python 3.9, Ubuntu 20.04, formats) |
| `doc/scapy/conf.py` | 1–60 | Sphinx build configuration (extensions, autodoc settings, minimum Sphinx version) |
| `doc/scapy/index.rst` | Full file | Documentation table of contents and section organization |
| `doc/scapy/layers/index.rst` | Full file | Layer-specific documentation index (glob pattern for layer docs) |
| `doc/scapy/build_dissect.rst` | Lines 1085–1100 | Field type listings including DNS field types |
| `doc/scapy/usage.rst` | DNS-related lines (1354–1400) | Existing DNS usage examples (queries, spoofing) |
| `scapy/layers/llmnr.py` | 1–30 | LLMNR module that reuses DNS field types |
| `README.md` | Full file (via summary) | Project overview |

**Folders Explored:**

| Folder Path | Purpose |
|-------------|---------|
| Root (`""`) | Repository structure overview |
| `scapy/` | Core package structure |
| `scapy/layers/` | Protocol layer catalog |
| `doc/` | Documentation ecosystem |
| `doc/scapy/` | Sphinx documentation source |
| `doc/scapy/layers/` | Layer-specific documentation |
| `test/scapy/layers/` | Test file location |

**Semantic and Pattern Searches Conducted:**

| Search Type | Query / Pattern | Result |
|-------------|-----------------|--------|
| `find -iname "*dns*"` | DNS-related files | Found `scapy/layers/dns.py`, `test/scapy/layers/dns.uts`, `test/scapy/layers/dns_dnssec.uts`, `test/scapy/layers/dns_edns0.uts` |
| `grep "compress\|decompress"` | Compression-related code | Located all compression/decompression functions and test cases |
| `grep "_orig_s\|InheritOrigin"` | Cross-boundary resolution mechanism | Mapped all uses of the `_orig_s` attribute across DNS classes |
| `grep "DNSStrField"` across repository | Cross-module usage | Found reuse in `dcerpc.py`, `dhcp6.py`, `socks.py`, and inspiration in `pfcp.py`, `gtp.py` |
| `find -name "*.md" -o -name "*.rst"` | Documentation files | Cataloged all existing documentation files |

**Tech Spec Sections Retrieved:**

| Section | Purpose |
|---------|---------|
| 1.1 Executive Summary | Project overview, version, license, Python requirements |
| 2.1 Feature Catalog | Feature F-004 (Built-in Protocol Stack) confirming `dns.py` role |
| 4.2 Core Packet Engine Processes | Packet dissection lifecycle context for DNS field extraction |
| 8.6 Documentation Infrastructure | Documentation tooling (Sphinx, RTD) configuration details |

### 0.11.2 Attachments

No attachments were provided by the user. No Figma URLs or external design assets are referenced in this task.

### 0.11.3 External References

No external web searches were conducted. All documentation content is derived from direct source code analysis of the Scapy repository at commit `0925ada4` on branch `scapy_0925ada48540`.


