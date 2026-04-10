# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

### 0.1.1 Core Documentation Objective

Based on the provided requirements, the Blitzy platform understands that the documentation objective is to **create a new investigative Q&A markdown document** that comprehensively explains Scapy's internal checksum computation, field auto-computation, and raw packet caching mechanisms through five concrete, hands-on scenarios with full hex output evidence.

- **Category:** Create new documentation
- **Documentation type:** Technical Q&A / Investigative explainer document
- **Target audience:** Developers and network engineers working with Scapy who need to understand when and how fields like IP checksum, TCP checksum, and IP length are computed, cached, and overridden

The user's requirements decompose into five distinct investigation scenarios:

- **Scenario 1 — Initial Checksum Computation:** Build an `IP()/TCP()/Raw(load=...)` packet without explicitly setting checksum fields, call `bytes()`, and display the computed IP header checksum and TCP checksum as hex values at their exact byte positions
- **Scenario 2 — Payload Modification and Recalculation:** After the initial build, modify a byte in the payload and call `bytes()` again; determine whether Scapy returns fresh checksums or stale cached bytes; show complete before/after hex dumps for byte-by-byte comparison
- **Scenario 3 — Manual Checksum Override (Before vs. After Build):** Assign `IP.chksum = 0xAAAA` before ever building the packet, then build it; separately, build the packet first and then assign `0xAAAA`; determine whether the manual value survives into final bytes in each case
- **Scenario 4 — Deep Copy Isolation:** Build a packet to bytes, make a `copy.deepcopy()` of the packet, modify the copy's payload, and confirm the original packet's bytes remain unaffected
- **Scenario 5 — IP Length Field Consistency After Growth:** Build a packet with a small payload, then append ten more bytes and rebuild; report the IP length field value (decimal), the actual byte count, and whether they match

### 0.1.2 Special Instructions and Constraints

- **No source code modifications:** The user explicitly states "don't modify the code." This is reinforced by the implementation rule: "Do not modify any existing files in the source repository."
- **Temporary cleanup:** The user requests "clean up any temporary stuff when you're done." Any test scripts are exploratory only and must not persist.
- **Output file naming:** Per the `SWE-AtlasQnA-Repo` rule, the document must be named `scapy_0925ada48540.md` (matching the source branch name) and placed in the `blitzy/documentation/` directory.
- **Evidence-based answers:** Per the implementation rule: "Do not make assumptions, base your answers on the code as the truth."
- **Thinking/rationale required:** Per the implementation rule: "Provide thinking / rationale behind the answers."
- **Full hex dumps required:** The user insists on "actual hex dumps and numeric values" — not just diffs but complete before-and-after outputs for comparison.
- **Test script exploration:** The user says "Feel free to write test scripts to explore this" — scripts may be used to generate evidence but must not modify the Scapy source.

### 0.1.3 Technical Interpretation

These documentation requirements translate to the following technical documentation strategy:

- To **document checksum computation mechanics**, we will create a new file `blitzy/documentation/scapy_0925ada48540.md` that explains the `post_build()` mechanism in `scapy/layers/inet.py` (IP class at line 539, TCP class at line 767) where checksums are computed only when `self.chksum is None`
- To **document caching behavior**, we will trace the `raw_packet_cache` attribute lifecycle defined in `scapy/packet.py` (initialization at line 169, cache check in `self_build()` at line 685, cache invalidation in `setfieldval()` at line 485)
- To **document manual override behavior**, we will explain how `setfieldval()` (line 472) stores the user-assigned value, which causes `post_build()` to skip recomputation because `self.chksum is None` evaluates to `False`
- To **document deep copy isolation**, we will reference the `__deepcopy__()` method at line 239 (which delegates to `copy()` at line 407) that creates independent field dictionaries and payload chains
- To **document IP length auto-computation**, we will explain the `post_build()` logic in `IP` (line 545: `if self.len is None`) which computes `len(p) + len(pay)` at build time

### 0.1.4 Inferred Documentation Needs

Based on code analysis, the following implicit documentation needs are surfaced:

- **Constructed vs. dissected packet distinction:** Constructed packets (created via `IP()/TCP()`) have `chksum=None` by default, meaning `post_build()` always recomputes checksums. Dissected packets (created via `IP(raw_bytes)`) have explicit field values from the wire, meaning `post_build()` will NOT recompute unless the field is explicitly deleted. This critical distinction is undocumented and must be explained.
- **Cache scope limitation:** The `raw_packet_cache` is only populated during dissection (`do_dissect()` at line 1019), never during construction. For constructed packets, `raw_packet_cache` remains `None`, so `self_build()` always rebuilds from field values. This means caching is not a concern for the user's scenarios involving constructed packets.
- **Cache invalidation boundary:** When a deeper layer's field is modified (e.g., `Raw.load`), only that layer's cache is cleared. Parent layer caches (IP, TCP) are NOT automatically invalidated. This matters for dissected packets and should be documented as a potential pitfall.
- **The `clear_cache()` method:** Users working with dissected packets who modify inner layers should call `clear_cache()` and `del pkt[IP].chksum` / `del pkt[TCP].chksum` to force full recomputation. This workflow must be documented.

## 0.2 Documentation Discovery and Analysis

### 0.2.1 Existing Documentation Infrastructure Assessment

Repository analysis reveals a mature Sphinx-based documentation ecosystem with partial coverage of the checksum and caching topics the user is asking about.

- **Documentation framework:** Sphinx `>=3.0.0` with `sphinx_rtd_theme>=0.4.3`, configured in `doc/scapy/conf.py`
- **Documentation generator configuration:** `doc/scapy/conf.py` — sets `sys.path` to the repo root, uses extensions `sphinx.ext.autodoc`, `sphinx.ext.napoleon`, `sphinx.ext.todo`, `sphinx.ext.linkcode`, and custom `scapy_doc`
- **Hosted documentation:** ReadTheDocs at `https://scapy.readthedocs.io`, configured via `.readthedocs.yml` (Ubuntu 20.04, Python 3.9, `pip install .[docs]`)
- **API documentation tools:** `sphinx-apidoc` via the `apitree` tox environment regenerates API docs from Python docstrings
- **Diagram tools detected:** No Mermaid integration in the existing Sphinx docs; diagrams are static images in `doc/scapy/graphics/`
- **Output formats:** HTML (default), EPUB, PDF
- **`blitzy/documentation/` directory:** Does not yet exist in the repository; must be created

**Existing documentation files inspected:**

| File | Content Summary | Relevance to User's Question |
|------|----------------|------------------------------|
| `doc/scapy/build_dissect.rst` | Explains `build()`, `do_build()`, `post_build()` lifecycle for protocol developers; shows how checksums are computed via `post_build()` | **High** — covers the build pipeline conceptually but lacks hex-level demonstration of caching behavior |
| `doc/scapy/usage.rst` | Interactive tutorial showing `show2()` for assembled packets, mentions "checksum is calculated" at line 171 | **Medium** — references checksums in passing but does not explain the caching mechanism or manual override |
| `README.md` | Project overview, installation, quick start | **Low** — no checksum or caching information |
| `CONTRIBUTING.md` | Contributor guide: PR expectations, testing, logging | **Low** — no technical packet engine documentation |
| `doc/notebooks/Scapy in 15 minutes.ipynb` | Interactive walkthrough of packet construction, layer stacking, send/receive | **Low** — demonstrates `show2()` but does not explore checksums or caching |

### 0.2.2 Repository Code Analysis for Documentation

The following source files were analyzed to extract the technical truth for the documentation:

**Primary sources for checksum computation:**

| File | Lines | Content |
|------|-------|---------|
| `scapy/layers/inet.py:521-551` | IP class definition and `post_build()` | IP checksum computed at line 549 via `checksum(p)` only when `self.chksum is None`; IP length computed at line 546 only when `self.len is None` |
| `scapy/layers/inet.py:753-786` | TCP class definition and `post_build()` | TCP checksum computed at line 777 via `in4_chksum()` only when `self.chksum is None` |
| `scapy/layers/inet.py:692-704` | `in4_chksum()` function | Computes IPv4 pseudo-header checksum per RFC 793 |
| `scapy/utils.py:496-504` | `checksum()` function | Standard one's complement checksum: sum 16-bit words, fold carry, bitwise NOT |

**Primary sources for caching behavior:**

| File | Lines | Content |
|------|-------|---------|
| `scapy/packet.py:169-170` | `__init__()` | `raw_packet_cache` initialized to `None`; `raw_packet_cache_fields` initialized to `None` |
| `scapy/packet.py:678-713` | `self_build()` | Checks `raw_packet_cache` validity by comparing cached field values; returns cached bytes if valid |
| `scapy/packet.py:724-740` | `do_build()` | If `raw_packet_cache` is `None`, calls `post_build()`; otherwise concatenates cached header + payload |
| `scapy/packet.py:1002-1021` | `do_dissect()` | Sets `raw_packet_cache` to the raw bytes consumed during dissection (line 1019); sets `explicit = 1` (line 1020) |
| `scapy/packet.py:472-486` | `setfieldval()` | Any field assignment clears `raw_packet_cache` (line 485) and sets `explicit = 0` (line 484) |
| `scapy/packet.py:664-676` | `clear_cache()` | Recursively clears `raw_packet_cache` for the layer, all packet-holding fields, and the payload chain |

**Primary sources for copy behavior:**

| File | Lines | Content |
|------|-------|---------|
| `scapy/packet.py:239-244` | `__deepcopy__()` | Delegates to `self.copy()` |
| `scapy/packet.py:407-425` | `copy()` | Creates a new instance, deep-copies `fields` and `default_fields` dicts, copies `raw_packet_cache`, recursively copies payload via `self.payload.copy()` |

### 0.2.3 Web Search Research Conducted

No web search was required for this task. All answers are derivable from the Scapy source code itself, which the implementation rules require: "base your answers on the code as the truth." The existing Sphinx documentation at `doc/scapy/build_dissect.rst` and the source code in `scapy/packet.py` and `scapy/layers/inet.py` provide complete evidence for all five scenarios.

## 0.3 Documentation Scope Analysis

### 0.3.1 Code-to-Documentation Mapping

The documentation must be grounded in the following code modules and their specific mechanisms:

- **Module: `scapy/packet.py` (Core Packet Engine)**
  - Public APIs relevant: `build()`, `do_build()`, `self_build()`, `post_build()`, `__bytes__()`, `copy()`, `__deepcopy__()`, `clear_cache()`, `setfieldval()`
  - Current documentation: Partially covered in `doc/scapy/build_dissect.rst` — explains the build lifecycle conceptually but omits caching semantics and hex-level examples
  - Documentation needed: Detailed explanation of `raw_packet_cache` lifecycle, interaction between field assignment and cache invalidation, behavior differences between constructed and dissected packets

- **Module: `scapy/layers/inet.py` (IP/TCP Protocol Layers)**
  - Public APIs relevant: `IP.post_build()` (line 539), `TCP.post_build()` (line 767), `in4_chksum()` (line 692)
  - Current documentation: Checksum computation mentioned in `doc/scapy/usage.rst` line 171 as a one-liner ("checksum is calculated, for instance") and in `build_dissect.rst` line 525 ("checksums are computed")
  - Documentation needed: Precise hex-level walkthrough of IP and TCP checksum byte positions, the `if self.chksum is None` guard, behavior when manually overriding

- **Module: `scapy/utils.py` (Checksum Algorithm)**
  - Public APIs relevant: `checksum()` (line 496)
  - Current documentation: Not documented in user-facing docs
  - Documentation needed: Brief explanation of the one's complement checksum algorithm as context for the IP/TCP checksum values

- **Configuration options requiring documentation:**
  - `Packet.raw_packet_cache` — per-instance attribute, not in `conf`
  - `Packet.explicit` — per-instance flag controlling auto-resolution
  - These are internal attributes, not conf options, but their behavior must be documented

### 0.3.2 Documentation Gap Analysis

Given the requirements and repository analysis, documentation gaps include:

- **Undocumented behavior — constructed vs. dissected caching:** No existing documentation explains that `raw_packet_cache` is only populated during dissection (`do_dissect()` at line 1019), while constructed packets always have `raw_packet_cache = None` and are rebuilt from scratch on every `bytes()` call
- **Undocumented behavior — parent cache not invalidated by child changes:** When modifying `pkt[Raw].load` on a dissected packet, only the Raw layer's cache is cleared; the IP and TCP layers retain their cached bytes, meaning `post_build()` is skipped and checksums are NOT recomputed. This is a significant gap.
- **Missing hex-level checksum examples:** Existing docs show `chksum=0x7ce7` in show output but never demonstrate the actual byte positions (bytes 10-11 for IP, bytes 36-37 for TCP in a standard 20-byte IP + 20-byte TCP header)
- **Missing manual override documentation:** No existing docs explain that assigning `pkt.chksum = 0xAAAA` causes `post_build()` to skip recomputation because the `if self.chksum is None` guard evaluates to `False`
- **Missing deep copy behavior documentation:** The `copy()` method documentation in source docstrings says only "Returns a deep copy of the instance" — no explanation of isolation guarantees for payload modifications
- **Missing IP length auto-computation examples:** The `if self.len is None` guard in `IP.post_build()` is not documented with concrete before/after examples showing length growth

## 0.4 Documentation Implementation Design

### 0.4.1 Documentation Structure Planning

The output is a single comprehensive markdown file placed in the `blitzy/documentation/` directory:

```text
blitzy/
└── documentation/
    └── scapy_0925ada48540.md
```

The document will be structured as follows:

```text
scapy_0925ada48540.md
├── Title and Introduction
│   ├── Question summary
│   └── Key concepts overview (post_build, raw_packet_cache, checksum guards)
├── Scenario 1: Initial Checksum Computation
│   ├── Rationale / code walkthrough
│   ├── Test script
│   └── Full hex dump with annotated byte positions
├── Scenario 2: Payload Modification and Recalculation
│   ├── Rationale / code walkthrough
│   ├── Test script
│   ├── Before hex dump
│   └── After hex dump with comparison
├── Scenario 3: Manual Checksum Override
│   ├── Scenario 3a: Set before building
│   ├── Scenario 3b: Set after building
│   ├── Rationale for both behaviors
│   └── Hex evidence for each
├── Scenario 4: Deep Copy Isolation
│   ├── Rationale / code walkthrough
│   ├── Test script
│   └── Before/after hex dumps proving isolation
├── Scenario 5: IP Length Field Growth
│   ├── Rationale / code walkthrough
│   ├── Test script
│   └── Decimal and hex evidence for length consistency
├── Summary of Key Findings
│   └── Table of behaviors across all scenarios
└── Source Citations
    └── File paths and line numbers
```

### 0.4.2 Content Generation Strategy

**Information Extraction Approach:**

- Extract the `post_build()` logic from `scapy/layers/inet.py:539-551` (IP) and `scapy/layers/inet.py:767-786` (TCP) to explain the `if self.chksum is None` guard
- Extract the `self_build()` cache-check logic from `scapy/packet.py:685-695` to explain when caching applies
- Extract the `setfieldval()` cache-invalidation logic from `scapy/packet.py:484-486` to explain why field assignments clear the cache
- Generate hex outputs by running actual Scapy code to produce deterministic, verifiable results
- Extract the `copy()` method logic from `scapy/packet.py:407-425` to explain isolation guarantees

**Documentation Standards:**

- Markdown formatting with `#`, `##`, `###` headers
- Code examples using fenced code blocks with language syntax highlighting
- Hex dumps formatted as monospaced text with byte-position annotations
- Source citations as inline references, e.g., `Source: scapy/packet.py:685`
- Tables for summary comparisons
- Mermaid diagrams for the build lifecycle flow

### 0.4.3 Diagram and Visual Strategy

The document will include the following Mermaid diagram:

- **Packet Build Decision Flow:** A flowchart showing the `build()` → `do_build()` → `self_build()` → `post_build()` pathway, with decision points for `raw_packet_cache` validity and `self.chksum is None` checks — this directly illustrates why checksums are or are not recomputed in each scenario

No screenshots or static images are required; all evidence is in hex dumps and code snippets.

## 0.5 Documentation File Transformation Mapping

### 0.5.1 File-by-File Documentation Plan

| Target Documentation File | Transformation | Source Code/Docs | Content/Changes |
|---------------------------|----------------|------------------|-----------------|
| `blitzy/documentation/scapy_0925ada48540.md` | CREATE | `scapy/packet.py`, `scapy/layers/inet.py`, `scapy/utils.py` | Comprehensive Q&A document answering all five scenarios about checksum computation, caching, manual override, deep copy isolation, and IP length auto-computation with full hex dumps, rationale, and source citations |

**Transformation Mode: CREATE** — This is the only file to be produced. No existing files are modified, updated, or deleted.

### 0.5.2 New Documentation File Detail

**File: `blitzy/documentation/scapy_0925ada48540.md`**

- **Type:** Technical Q&A / Investigative explainer
- **Source Code:**
  - `scapy/packet.py` — `build()` (line 746), `do_build()` (line 724), `self_build()` (line 678), `post_build()` (line 758), `__bytes__()` (line 592), `setfieldval()` (line 472), `copy()` (line 407), `__deepcopy__()` (line 239), `clear_cache()` (line 664), `do_dissect()` (line 1002)
  - `scapy/layers/inet.py` — `IP` class (line 521), `IP.post_build()` (line 539), `TCP` class (line 753), `TCP.post_build()` (line 767), `in4_chksum()` (line 692)
  - `scapy/utils.py` — `checksum()` (line 496)
- **Sections:**
  - Introduction: Overview of the five questions and key Scapy internals concepts
  - Scenario 1: Initial checksum computation with full hex output and annotated byte positions (IP bytes 10-11, TCP bytes 36-37)
  - Scenario 2: Payload modification and checksum recalculation — before/after hex dumps proving TCP checksum changes while IP checksum stays the same (payload change does not alter IP header fields)
  - Scenario 3a: Manual checksum set before building — proving `0xAAAA` survives because `post_build()` skips computation when `self.chksum is not None`
  - Scenario 3b: Manual checksum set after building — proving `0xAAAA` also survives because `setfieldval()` clears cache and stores the value, then `post_build()` sees it as non-None
  - Scenario 4: Deep copy isolation — before/after hex dumps proving `copy.deepcopy()` creates fully independent packet hierarchies via `Packet.copy()` at line 407
  - Scenario 5: IP length auto-computation — showing small payload (45 bytes) and grown payload (55 bytes) with matching IP.len field values
  - Summary: Table of all scenario results
  - Source Citations: All file paths and line numbers referenced
- **Diagrams:**
  - Mermaid flowchart of the `build()` → `do_build()` → `self_build()` → `post_build()` pathway showing cache check and checksum computation decision points
- **Key Citations:** `scapy/packet.py`, `scapy/layers/inet.py`, `scapy/utils.py`

### 0.5.3 Documentation Configuration Updates

No documentation configuration updates are required. The new file is a standalone markdown document placed in `blitzy/documentation/`. It is not part of the Sphinx documentation tree (`doc/scapy/`) and does not require changes to:
- `doc/scapy/conf.py`
- `.readthedocs.yml`
- `tox.ini` documentation environments
- `pyproject.toml` docs extras

### 0.5.4 Cross-Documentation Dependencies

- **No shared content/includes:** The new document is self-contained
- **No navigation links:** The document is not integrated into the Sphinx toctree
- **No table of contents updates:** Not applicable
- **No index/glossary updates:** Not applicable

## 0.6 Dependency Inventory

### 0.6.1 Documentation Dependencies

The following packages are required to run the exploratory test scripts that generate the hex evidence documented in the output file:

| Registry | Package Name | Version | Purpose |
|----------|--------------|---------|---------|
| pip | scapy | 2.7.0 (commit `0925ada4`) | Core packet manipulation library being documented; installed from local repo in editable mode |
| pip | setuptools | >=62.0.0 | Build backend required by `pyproject.toml` for editable install of Scapy |

No additional documentation tooling is required because:
- The output is a plain markdown file, not a Sphinx-built document
- Mermaid diagrams are embedded as fenced code blocks (rendered by GitHub/readers natively)
- No documentation site build or deployment is performed

### 0.6.2 Runtime Environment

| Component | Version | Source |
|-----------|---------|--------|
| Python | 3.12.3 | System installation; project requires `>=3.7, <4` per `pyproject.toml` |
| Operating System | Linux (Ubuntu) | Required for Scapy's default `L3PacketSocket` behavior, though the documented scenarios only use in-memory packet construction (no network I/O) |

### 0.6.3 Documentation Reference Updates

Not applicable — no existing documentation links need to be updated. The new file `blitzy/documentation/scapy_0925ada48540.md` is a standalone addition with no inbound or outbound links to the existing Sphinx documentation tree.

## 0.7 Coverage and Quality Targets

### 0.7.1 Documentation Coverage Metrics

**Current coverage analysis of the user's five scenarios:**

| Scenario | Existing Docs Coverage | Target Coverage |
|----------|----------------------|-----------------|
| 1. Initial checksum computation | Partial — `build_dissect.rst` line 525 mentions "checksums are computed" but no hex evidence | 100% — full hex dump with byte-position annotations |
| 2. Payload modification and recalculation | None — no existing docs address this | 100% — before/after hex dumps proving fresh recomputation |
| 3. Manual checksum override (before/after) | None — no existing docs explain the `if self.chksum is None` guard behavior | 100% — both sub-scenarios with hex evidence and code rationale |
| 4. Deep copy isolation | None — `copy()` docstring says only "Returns a deep copy" | 100% — hex dumps proving complete isolation |
| 5. IP length field growth | None — no existing docs demonstrate length auto-computation with examples | 100% — decimal and hex evidence showing consistency |

**Target coverage:** 100% of all five user-specified scenarios, each with:
- Code-based rationale citing specific source file paths and line numbers
- Reproducible test script
- Full hex dump output (not just diffs)

### 0.7.2 Documentation Quality Criteria

**Completeness requirements:**

- Every scenario must include: (a) the underlying Scapy source code logic, (b) a runnable Python script demonstrating the behavior, (c) the actual hex output from running that script, and (d) an interpretation explaining which bytes changed and why
- The introduction must establish the three key concepts: `post_build()`, `raw_packet_cache`, and the `if self.chksum is None` guard
- The summary must consolidate all findings into a single comparison table

**Accuracy validation:**

- All hex outputs were generated by running actual Scapy code in the repository environment (Python 3.12.3, Scapy commit `0925ada4`)
- Code rationale is traceable to specific line numbers in the source
- The document must not contain assumptions — all claims must be backed by source code evidence per the implementation rule

**Clarity standards:**

- Technical accuracy with accessible language for developers familiar with Python but not necessarily Scapy internals
- Progressive disclosure: start with the simplest scenario (initial build) and build toward more complex scenarios (caching, overrides)
- Consistent hex dump formatting with byte-position annotations

**Maintainability:**

- Source citations include file paths and line numbers for traceability
- All test scripts are self-contained and can be re-run to verify results

### 0.7.3 Example and Diagram Requirements

- **Minimum examples per scenario:** 1 complete Python script with full hex output
- **Diagram types required:** 1 Mermaid flowchart (build lifecycle with checksum decision points)
- **Code example testing:** All examples are verified by execution against the actual Scapy codebase in the repository
- **Hex output format:** Full hex strings with annotated byte offsets for checksum and length fields

## 0.8 Scope Boundaries

### 0.8.1 Exhaustively In Scope

- **New documentation files:**
  - `blitzy/documentation/scapy_0925ada48540.md` — the sole deliverable

- **Source code analyzed (read-only) for documentation content:**
  - `scapy/packet.py` — `build()`, `do_build()`, `self_build()`, `post_build()`, `__bytes__()`, `setfieldval()`, `copy()`, `__deepcopy__()`, `clear_cache()`, `do_dissect()`, `__init__()`
  - `scapy/layers/inet.py` — `IP` class, `IP.post_build()`, `TCP` class, `TCP.post_build()`, `in4_chksum()`, `in4_pseudoheader()`
  - `scapy/utils.py` — `checksum()`
  - `scapy/fields.py` — `XShortField` (used for `chksum` field definition)

- **Topics covered in the documentation:**
  - IP header checksum computation via `post_build()` at byte positions 10-11
  - TCP checksum computation via `post_build()` with IPv4 pseudo-header at byte positions 36-37
  - The `raw_packet_cache` lifecycle: initialization, population during dissection, invalidation on field assignment
  - Difference between constructed packets (`chksum=None`) and dissected packets (`chksum=<wire value>`)
  - Manual checksum override behavior (pre-build and post-build assignment)
  - `copy.deepcopy()` isolation via `Packet.copy()` method
  - IP length field (`IP.len`) auto-computation and consistency after payload growth
  - The `clear_cache()` method and `del pkt[IP].chksum` pattern for dissected packet recomputation

### 0.8.2 Explicitly Out of Scope

- **Source code modifications:** No changes to any `.py` file in the Scapy source tree — enforced by the user directive ("don't modify the code") and the `SWE-AtlasQnA-Repo` rule ("Do not modify any existing files")
- **Test file modifications:** No changes to `test/` directory files
- **Existing documentation modifications:** No changes to `doc/scapy/*.rst`, `README.md`, `CONTRIBUTING.md`, or any other existing documentation file
- **Feature additions or code refactoring:** Not applicable
- **Deployment configuration changes:** No changes to `.readthedocs.yml`, `tox.ini`, `pyproject.toml`, or CI configurations
- **Sphinx documentation build:** The new file is not integrated into the Sphinx build; no `doc/scapy/conf.py` or toctree changes
- **IPv6 checksum behavior:** The user's scenarios use only IPv4; IPv6 pseudo-header checksum (`in6_chksum`) is not in scope
- **UDP checksum behavior:** The user asks only about IP and TCP; UDP's `post_build()` at `scapy/layers/inet.py:841` is not in scope
- **Network I/O or transmission:** All scenarios involve in-memory packet construction and serialization only; no `send()`, `sniff()`, or socket operations
- **Scapy configuration tuning:** No changes to `conf` singleton settings
- **Persistent test scripts:** The user requests "clean up any temporary stuff when you're done" — any exploratory scripts used to generate evidence are ephemeral and not delivered

## 0.9 Execution Parameters

### 0.9.1 Documentation-Specific Instructions

- **Documentation build command:** Not applicable — the deliverable is a standalone markdown file, not a Sphinx-built document
- **Documentation preview command:** Any markdown viewer or `cat blitzy/documentation/scapy_0925ada48540.md`
- **Diagram generation command:** Not applicable — Mermaid diagrams are embedded as fenced code blocks and rendered by the reader's environment (GitHub, VS Code, etc.)
- **Documentation deployment command:** Not applicable — no deployment step required
- **Default format:** Markdown with Mermaid diagrams
- **Citation requirement:** Every technical claim must reference the source file and line number (e.g., `Source: scapy/packet.py:685`)
- **Style guide:** Follow the `SWE-AtlasQnA-Repo` rule — provide thinking/rationale behind answers, base answers on the code as truth, do not make assumptions
- **Documentation validation:** Verify hex outputs are reproducible by re-running the test scripts against the Scapy installation

### 0.9.2 Evidence Generation Parameters

The hex dumps and numeric values documented in the output file are generated using the following deterministic setup:

| Parameter | Value |
|-----------|-------|
| Source IP | `169.254.9.1` (auto-resolved by Scapy for `dst=10.0.0.1`) |
| Destination IP | `10.0.0.1` |
| TCP source port | 20 (default) |
| TCP destination port | 80 |
| Payload | `b"HELLO"` (5 bytes, ASCII) |
| IP header length | 20 bytes (no options) |
| TCP header length | 20 bytes (no options) |
| Total packet size | 45 bytes (20 + 20 + 5) |

These parameters ensure consistent, reproducible hex output across runs on the same Scapy version. The source IP may vary across environments, so the documentation will note that byte positions 12-15 (source IP) and the resulting checksum values may differ on other machines.

## 0.10 Rules for Documentation

The following rules are explicitly specified by the user or derived from the implementation rules and must be strictly followed:

- **"Do not modify any existing files in the source repository"** — No changes to any file in the Scapy codebase. The only write operation is creating the new file `blitzy/documentation/scapy_0925ada48540.md`.
- **"Do not make assumptions, base your answers on the code as the truth"** — Every claim about checksum computation, caching behavior, copy isolation, and length auto-computation must be traceable to specific source code locations. No speculative or hypothetical explanations.
- **"Provide thinking / rationale behind the answers"** — Each scenario must include not just the hex output but a clear explanation of WHY Scapy produces that output, grounded in the `post_build()`, `self_build()`, `setfieldval()`, and `copy()` implementations.
- **"Create a new markdown document named `<source_branch_name>.md`"** — The file must be named exactly `scapy_0925ada48540.md` (matching the branch name `scapy_0925ada48540`).
- **"Place the generated document in the `blitzy/documentation` directory"** — The directory must be created if it does not exist.
- **"Feel free to write test scripts to explore this, just don't modify the code and clean up any temporary stuff when you're done"** — Test scripts are used internally to generate evidence but are not part of the deliverable. Any temporary files must be removed after evidence generation.
- **"Show the before and after hex outputs explicitly, not just the differences"** — Full hex dumps must be provided for every scenario, enabling side-by-side comparison at the reader's discretion.
- **"I need to see the actual hex dumps and numeric values"** — No abstracted or summarized outputs. Raw hex strings and decimal values must be presented for IP checksum, TCP checksum, IP length, and total byte count.

## 0.11 References

### 0.11.1 Source Code Files Searched and Analyzed

| File Path | Lines Inspected | Purpose |
|-----------|----------------|---------|
| `scapy/packet.py` | 85-100, 155-185, 239-244, 400-425, 465-520, 580-770, 990-1030 | Core packet engine: `build()`, `do_build()`, `self_build()`, `post_build()`, `__bytes__()`, `setfieldval()`, `delfieldval()`, `copy()`, `__deepcopy__()`, `clear_cache()`, `do_dissect()`, `raw_packet_cache` lifecycle, `explicit` flag |
| `scapy/layers/inet.py` | 521-560, 667-704, 753-800 | IP class definition, `IP.post_build()` (checksum, length, IHL), TCP class definition, `TCP.post_build()` (TCP checksum via pseudo-header), `in4_chksum()`, `in4_pseudoheader()` |
| `scapy/utils.py` | 486-545 | `checksum()` function (one's complement), `checksum_endian_transform`, `fletcher16_checksum()` |
| `scapy/fields.py` | (summary only) | Field type system: `XShortField` (used for `chksum`), `ShortField` (used for `len`), `BitField` (used for `ihl`, `version`) |
| `pyproject.toml` | Full file | Project metadata, Python version requirement (`>=3.7, <4`), optional dependency groups, build system configuration |
| `setup.py` | Full file | Legacy build script, VERSION file generation |
| `tox.ini` | Lines 1-50 | Test matrix (Python 3.7-3.11), documentation build environments |
| `.readthedocs.yml` | Full file | ReadTheDocs hosting configuration (Python 3.9, Ubuntu 20.04) |
| `doc/scapy/conf.py` | Lines 1-50 | Sphinx build configuration, extensions, version metadata |
| `doc/scapy/build_dissect.rst` | Lines 520-670 | Existing documentation on `build()`, `post_build()`, checksum computation, field serialization |
| `doc/scapy/usage.rst` | grep results | Existing documentation references to checksums (line 171: "checksum is calculated") |
| `README.md` | Lines 1-80 | Project overview, platform support, quick start |

### 0.11.2 Folders Searched

| Folder Path | Purpose |
|-------------|---------|
| (root) | Repository structure, top-level configuration files |
| `scapy/` | Core runtime package, all first-order children |
| `scapy/layers/` | Protocol layer modules including `inet.py` |
| `doc/` | Documentation ecosystem |
| `doc/scapy/` | Sphinx manual source tree |

### 0.11.3 Tech Spec Sections Consulted

| Section | Relevance |
|---------|-----------|
| 1.1 Executive Summary | Project context, version info (2.7.0), stakeholder identification |
| 1.3 Scope | In-scope/out-of-scope boundaries, platform coverage |
| 3.3 Frameworks & Libraries | Build system, zero-dependency design, optional dependency groups |
| 4.2 Core Packet Engine Processes | Packet build lifecycle, cache check flow, `post_build()` hook documentation |
| 5.2 Component Details | Core packet engine architecture, field API, `raw_packet_cache` description |
| 8.6 Documentation Infrastructure | Sphinx configuration, ReadTheDocs setup, API doc generation |

### 0.11.4 Attachments

No attachments were provided for this project. The user's request was entirely text-based, describing five investigative scenarios about Scapy's checksum and caching behavior.

