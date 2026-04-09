# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

### 0.1.1 Core Documentation Objective

Based on the provided requirements, the Blitzy platform understands that the documentation objective is to **create new documentation** that comprehensively answers a series of targeted technical questions about Scapy's internal packet-routing and ARP-resolution behavior. This is a **deep-code-analysis Q&A document**, not a high-level user guide or API reference.

**Category:** Create new documentation
**Documentation Type:** Technical Q&A / Code-Driven Investigation Report

The user is onboarding to the Scapy codebase and seeks authoritative, code-backed answers to these specific questions:

- **Q1 — Interface Selection:** How does Scapy determine which network interface to use when a packet is sent without an explicit interface parameter?
- **Q2 — MAC Address Resolution (ARP):** Once an interface and gateway are identified, how does Scapy resolve the next-hop MAC address for reaching an external destination?
- **Q3 — ARP Caching on Repeated Sends:** When the same packet is sent twice in quick succession, what ARP behavior is observed? Does Scapy re-ARP or use a cache?
- **Q4 — No-Route Error Handling:** What happens if a packet is sent to a destination with no matching route? At what point does the failure occur, and what error message is produced?

Additionally, the user requests:
- Setting up Scapy from the repository and starting an interactive session (environment validation)
- Permission to run temporary scripts for testing, but the actual codebase must remain unchanged

### 0.1.2 Special Instructions and Constraints

- **CRITICAL: Do not modify any existing files in the source repository** — this is explicitly stated both by the user ("leave the actual codebase unchanged") and by the implementation rule ("Do not modify any existing files in the source repository").
- **Code-as-truth principle:** All answers must be grounded in the actual source code, not assumptions or external documentation. As the implementation rule states: "Do not make assumptions, base your answers on the code as the truth."
- **Provide thinking/rationale:** Each answer must include the reasoning and code-path analysis behind the conclusions.
- **Output artifact:** A single markdown document named `scapy_0925ada48540.md` placed in the `blitzy/documentation` directory, as specified by the `SWE-AtlasQnA-Repo` implementation rule.
- **Temporary scripts are acceptable** for experimentation, but no persistent changes to the repository.

### 0.1.3 Technical Interpretation

These documentation requirements translate to the following technical documentation strategy:

- To **answer Q1 (Interface Selection)**, we will trace the code path from `send()` in `scapy/sendrecv.py` through `_interface_selection()`, `IP.route()` in `scapy/layers/inet.py`, and `Route.route()` in `scapy/route.py`, documenting how the routing table is consulted and how the most-specific-match algorithm selects the outgoing interface.
- To **answer Q2 (MAC Resolution)**, we will trace from `DestMACField.i2h()` in `scapy/layers/l2.py` through `conf.neighbor.resolve()`, the `inet_register_l3()` resolver, and `getmacbyip()`, documenting the ARP who-has broadcast and gateway indirection logic.
- To **answer Q3 (ARP Caching)**, we will document the `_arp_cache` CacheInstance (120-second TTL) in `scapy/layers/l2.py` and the `Route.cache` dictionary in `scapy/route.py`, explaining why the second rapid send produces no ARP traffic.
- To **answer Q4 (No-Route Error)**, we will trace the empty-match branch of `Route.route()`, the resulting loopback fallback, and the "No route found (no default route?)" warning message, as well as the `ScapyNoDstMacException` behavior when `conf.raise_no_dst_mac` is set.

### 0.1.4 Inferred Documentation Needs

Based on code analysis, the following implicit documentation needs have been identified:

- **Route table initialization** — Understanding where the routing data comes from (`/proc/net/route` on Linux, parsed in `scapy/arch/linux.py:read_routes()`) provides essential context for all four questions.
- **Two-tier caching architecture** — The interplay between `Route.cache` (route-level, no timeout, invalidated on table mutation) and `_arp_cache` (ARP-level, 120-second TTL) is central to Q3 but undocumented in existing Scapy docs.
- **conf.iface default selection** — The `get_working_if()` function in `scapy/interfaces.py` determines the fallback interface, which is a prerequisite for understanding Q1 fully.
- **Neighbor resolution framework** — The `Neighbor` class in `scapy/layers/l2.py` and its per-layer-pair resolver registration pattern (`conf.neighbor.register_l3()`) must be explained to answer Q2 completely.

## 0.2 Documentation Discovery and Analysis

### 0.2.1 Existing Documentation Infrastructure Assessment

Repository analysis reveals a **Sphinx-based documentation system** with moderate coverage of routing and send/receive operations, but no existing document that answers the user's specific code-path-tracing questions.

**Documentation framework:** Sphinx, version ≥ 3.0.0 (per `doc/scapy/conf.py:31`)
**Documentation generator configuration:** `doc/scapy/conf.py`
**Hosting/deployment:** Read the Docs (configured in `.readthedocs.yml`, building with Python 3.9 on Ubuntu 20.04)
**Theme:** `sphinx_rtd_theme` ≥ 0.4.3 (per `pyproject.toml` optional `docs` dependency)
**API documentation tools:** `sphinx.ext.autodoc`, `sphinx.ext.napoleon`, custom `scapy_doc` extension
**Diagram tools detected:** None in documentation build config (Mermaid not configured in Sphinx)

Existing documentation files examined:

| File | Relevance | Coverage |
|------|-----------|----------|
| `doc/scapy/routing.rst` | High — covers `conf.route`, `get_if_list()`, `getmacbyip()` | Surface-level API usage; does not explain internal code flow |
| `doc/scapy/usage.rst` | Medium — documents `send()`, `sendp()`, `sr()`, routing table manipulation | User-facing examples; no code-path tracing |
| `doc/scapy/index.rst` | Low — table of contents for the Sphinx manual | Structural reference only |
| `README.md` | Low — project overview and quick start | No routing internals |
| `doc/notebooks/Scapy in 15 minutes.ipynb` | Low — broad tutorial | Covers send/receive basics, not routing internals |

**Gap assessment:** No existing documentation traces the complete code path from `send(IP(dst="x.x.x.x"))` through interface selection, route lookup, neighbor resolution, ARP, to frame transmission. The user's questions require a new document.

### 0.2.2 Repository Code Analysis for Documentation

The following source files were examined to extract the routing and ARP behavior:

**Primary modules (directly implement the routing/ARP logic):**
- `scapy/route.py` — `Route` class: route table management, `route()` method with longest-prefix-match and metric tie-breaking, route cache
- `scapy/layers/l2.py` — `Neighbor` class, `DestMACField`, `SourceMACField`, `getmacbyip()`, `_arp_cache`, ARP packet definition, `inet_register_l3()` resolver registration
- `scapy/layers/inet.py` — `IP` class with `route()` method, `DestIPField`, `inet_register_l3()` linking Ether+IP to `getmacbyip()`
- `scapy/sendrecv.py` — `send()`, `sendp()`, `_interface_selection()`, `_send()`, `sr()`, `srp()`
- `scapy/config.py` — `conf` singleton, `NetCache`, `CacheInstance` (timeout-based cache), `raise_no_dst_mac`, `loopback_name`
- `scapy/interfaces.py` — `NetworkInterfaceDict`, `get_working_if()`, `resolve_iface()`

**Platform-specific modules (route table source):**
- `scapy/arch/__init__.py` — OS detection, platform-conditional imports, `get_if_hwaddr()`, `get_if_addr()`
- `scapy/arch/linux.py` — `read_routes()` parsing `/proc/net/route`, `LinuxInterfaceProvider`

**Error handling modules:**
- `scapy/error.py` — `Scapy_Exception`, `ScapyNoDstMacException`, `warning()` function, `log_runtime` logger

### 0.2.3 Web Search Research Conducted

No external web search was required for this task. All answers are derived directly from the Scapy source code, which the implementation rule mandates as the sole source of truth. The existing Sphinx documentation at `doc/scapy/routing.rst` and `doc/scapy/usage.rst` was consulted for style reference only.

## 0.3 Documentation Scope Analysis

### 0.3.1 Code-to-Documentation Mapping

The user's four questions map to the following modules and code paths:

**Q1 — Interface Selection (send without explicit interface):**

- Module: `scapy/sendrecv.py`
  - Public APIs: `send()` (line 422), `_interface_selection()` (line 616), `_send()` (line 396)
  - Current documentation: Partially documented in `doc/scapy/usage.rst` (user-facing) and `doc/scapy/routing.rst` (surface-level)
  - Documentation needed: Complete code-path trace from `send()` → `_interface_selection()` → `IP.route()` → `Route.route()`

- Module: `scapy/layers/inet.py`
  - Public APIs: `IP.route()` (line 559)
  - Current documentation: Not documented at the code-path level
  - Documentation needed: How IP.route() delegates to conf.route.route(dst)

- Module: `scapy/route.py`
  - Public APIs: `Route.route()` (line 146), `Route.__init__()` (line 32), `Route.resync()` (line 41)
  - Current documentation: `doc/scapy/routing.rst` shows usage but not algorithmic internals
  - Documentation needed: Longest-prefix-match algorithm, metric tie-breaking, cache behavior, route table source

- Module: `scapy/interfaces.py`
  - Public APIs: `get_working_if()` (line 350), `resolve_iface()` (line 386)
  - Current documentation: Partially in `doc/scapy/routing.rst`
  - Documentation needed: How `conf.iface` default is selected at startup

**Q2 — MAC Address Resolution (ARP for external destination):**

- Module: `scapy/layers/l2.py`
  - Public APIs: `getmacbyip()` (line 122), `DestMACField.i2h()` (line 166), `Neighbor.resolve()` (line 103)
  - Current documentation: `doc/scapy/routing.rst` shows `getmacbyip()` usage example only
  - Documentation needed: Full neighbor resolution chain, gateway indirection, ARP broadcast mechanism, `_arp_cache` with 120s TTL

- Module: `scapy/layers/inet.py`
  - Public APIs: `inet_register_l3()` (line 1125), registration at line 1129
  - Current documentation: Not documented
  - Documentation needed: How Ether+IP neighbor resolution is wired to `getmacbyip()`

**Q3 — ARP Caching on Repeated Sends:**

- Module: `scapy/config.py`
  - Public APIs: `CacheInstance` (line 350), `NetCache` (line 488)
  - Current documentation: Not documented
  - Documentation needed: TTL-based cache logic, `_timetable` timestamps, expiry check in `__getitem__`

- Module: `scapy/route.py`
  - Public APIs: `Route.cache` dict (line 39), `Route.invalidate_cache()` (line 37)
  - Current documentation: Not documented
  - Documentation needed: Route-level caching distinct from ARP-level caching

**Q4 — No-Route Error Handling:**

- Module: `scapy/route.py`
  - Public APIs: `Route.route()` (line 146) — the `if not paths:` branch at line 187
  - Current documentation: Not documented
  - Documentation needed: Warning message, loopback fallback, "0.0.0.0" return values

- Module: `scapy/error.py`
  - Public APIs: `ScapyNoDstMacException` (line 38), `warning()` (line 132)
  - Current documentation: Not documented
  - Documentation needed: Exception class and its relationship to `conf.raise_no_dst_mac`

- Module: `scapy/layers/l2.py`
  - Public APIs: `DestMACField.i2h()` (line 166) — the `conf.raise_no_dst_mac` branch
  - Current documentation: Not documented
  - Documentation needed: Broadcast fallback vs exception-raise behavior

### 0.3.2 Documentation Gap Analysis

Given the requirements and repository analysis, documentation gaps include:

- **Undocumented internal algorithms:** The `Route.route()` longest-prefix-match with metric tie-breaking is central to all four questions but has no dedicated documentation
- **Undocumented caching architecture:** Neither the route cache (`Route.cache`) nor the ARP cache (`_arp_cache` in `scapy/layers/l2.py`) are documented beyond their existence
- **Undocumented neighbor resolution framework:** The `Neighbor` class and its resolver-registration pattern are not explained in any existing documentation
- **Undocumented error paths:** The "No route found" warning, loopback fallback, and `ScapyNoDstMacException` behavior are absent from documentation
- **Missing consolidated flow documentation:** No single document connects the full path from `send()` through routing, neighbor resolution, ARP, to frame transmission

## 0.4 Documentation Implementation Design

### 0.4.1 Documentation Structure Planning

The output is a single comprehensive markdown document answering the user's questions with full code-path rationale:

```
blitzy/
└── documentation/
    └── scapy_0925ada48540.md
        ├── Introduction (context and environment setup)
        ├── Q1: Interface Selection Without Explicit Parameter
        │   ├── The send() Entry Point
        │   ├── _interface_selection() Logic
        │   ├── IP.route() Delegation
        │   ├── Route.route() — The Longest-Prefix-Match Algorithm
        │   ├── Route Table Source (/proc/net/route on Linux)
        │   └── conf.iface Fallback (get_working_if)
        ├── Q2: MAC Address Resolution for External Destinations
        │   ├── DestMACField Triggers Neighbor Resolution
        │   ├── Neighbor.resolve() and the Resolver Registry
        │   ├── inet_register_l3() → getmacbyip()
        │   ├── Gateway Indirection (ARP for gateway, not destination)
        │   ├── The ARP Broadcast (srp1 with who-has)
        │   └── ARP Cache Storage (_arp_cache with 120s TTL)
        ├── Q3: ARP Behavior on Rapid Repeated Sends
        │   ├── Two-Tier Caching: Route Cache + ARP Cache
        │   ├── First Send: Full Resolution Path
        │   ├── Second Send: Both Caches Hit
        │   └── Cache Expiry and Invalidation
        ├── Q4: Sending to an Unroutable Destination
        │   ├── Route.route() Empty-Match Branch
        │   ├── The Warning Message
        │   ├── Loopback Fallback and Broadcast MAC
        │   ├── conf.raise_no_dst_mac Behavior
        │   └── When the Failure Surfaces
        └── References (source files cited)
```

### 0.4.2 Content Generation Strategy

**Information Extraction Approach:**
- Extract routing algorithm from `scapy/route.py:Route.route()` (lines 146-197) using direct code reading
- Extract ARP resolution logic from `scapy/layers/l2.py:getmacbyip()` (lines 122-156) using direct code reading
- Extract cache behavior from `scapy/config.py:CacheInstance` (lines 350-485) using direct code reading
- Generate flow descriptions by tracing call chains across `sendrecv.py`, `inet.py`, `l2.py`, and `route.py`
- Validate conclusions by running Scapy interactively in the Python 3.11 virtual environment

**Documentation Standards:**
- Markdown formatting with headers `#`, `##`, `###`
- Code references in the format `Source: scapy/route.py:146-197`
- Short inline code snippets for key function signatures and critical lines
- Mermaid diagrams for the routing and ARP resolution flows
- Tables for parameter descriptions and cache characteristics
- Consistent use of Scapy terminology (e.g., "route tuple", "neighbor resolver", "cache instance")

### 0.4.3 Diagram and Visual Strategy

The documentation will include these Mermaid diagrams:

- **Interface Selection Flow** — A flowchart tracing from `send()` through `_interface_selection()` to `Route.route()`, showing the decision points and fallbacks
- **ARP Resolution Chain** — A flowchart from `DestMACField.i2h()` through `Neighbor.resolve()` and `getmacbyip()`, showing the gateway indirection and cache checks
- **Two-Tier Cache Architecture** — A diagram showing the relationship between `Route.cache` and `_arp_cache`, their TTL policies, and invalidation triggers
- **No-Route Error Path** — A flowchart showing the empty-match branch, warning emission, and loopback fallback

## 0.5 Documentation File Transformation Mapping

### 0.5.1 File-by-File Documentation Plan

| Target Documentation File | Transformation | Source Code/Docs | Content/Changes |
|---------------------------|----------------|------------------|-----------------|
| `blitzy/documentation/scapy_0925ada48540.md` | CREATE | `scapy/route.py`, `scapy/layers/l2.py`, `scapy/layers/inet.py`, `scapy/sendrecv.py`, `scapy/config.py`, `scapy/interfaces.py`, `scapy/error.py`, `scapy/arch/__init__.py`, `scapy/arch/linux.py` | Comprehensive Q&A document answering all four user questions with code-path rationale, Mermaid diagrams, and source citations |

**No other files are created, updated, or deleted.** The implementation rule explicitly prohibits modifying existing repository files.

### 0.5.2 New Documentation File Detail

```
File: blitzy/documentation/scapy_0925ada48540.md
Type: Technical Q&A / Investigation Report
Source Code:
  - scapy/sendrecv.py (send, _interface_selection, _send)
  - scapy/layers/inet.py (IP.route, inet_register_l3)
  - scapy/route.py (Route.route, Route.cache, Route.resync)
  - scapy/layers/l2.py (getmacbyip, _arp_cache, DestMACField, Neighbor)
  - scapy/config.py (CacheInstance, NetCache, conf.raise_no_dst_mac)
  - scapy/interfaces.py (get_working_if, resolve_iface)
  - scapy/error.py (ScapyNoDstMacException, warning)
  - scapy/arch/__init__.py (get_if_hwaddr, get_if_addr, platform dispatch)
  - scapy/arch/linux.py (read_routes from /proc/net/route)
Sections:
  - Introduction and Environment Setup
  - Q1: Interface Selection (routing code-path trace)
  - Q2: MAC Address Resolution (ARP and neighbor resolution trace)
  - Q3: ARP Caching Behavior on Rapid Sends (two-tier cache analysis)
  - Q4: No-Route Error Handling (error path and fallback analysis)
  - References
Diagrams:
  - Flowchart: Interface selection from send() to Route.route()
  - Flowchart: ARP resolution from DestMACField to getmacbyip()
  - Diagram: Two-tier caching (Route.cache + _arp_cache)
  - Flowchart: No-route error path
Key Citations:
  - scapy/route.py:146-197 (Route.route algorithm)
  - scapy/layers/l2.py:122-156 (getmacbyip)
  - scapy/layers/l2.py:161-179 (DestMACField.i2h)
  - scapy/sendrecv.py:616-631 (_interface_selection)
  - scapy/config.py:350-485 (CacheInstance)
  - scapy/error.py:38 (ScapyNoDstMacException)
```

### 0.5.3 Documentation Configuration Updates

No documentation configuration updates are required. The new file is placed in `blitzy/documentation/` which is an output directory separate from the existing Sphinx documentation tree at `doc/scapy/`. The existing `doc/scapy/conf.py`, `.readthedocs.yml`, and `tox.ini` documentation build configurations are not modified.

### 0.5.4 Cross-Documentation Dependencies

- **No shared content/includes:** The new document is standalone and self-contained
- **No navigation link updates:** The document is in `blitzy/documentation/`, not in the Sphinx `doc/scapy/` tree
- **No table of contents updates:** The document is independent of the existing Sphinx index
- **Reference to existing docs:** The document may reference `doc/scapy/routing.rst` and `doc/scapy/usage.rst` for context, but does not modify them

## 0.6 Dependency Inventory

### 0.6.1 Documentation Dependencies

The following tools and packages are relevant to building, running, and validating the documentation:

| Registry | Package Name | Version | Purpose |
|----------|--------------|---------|---------|
| pip | scapy | 2026.04.09 (from repo) | The target library being documented; installed in editable mode from repository |
| pip | setuptools | ≥ 62.0.0 | Build backend required by `pyproject.toml` |
| pip | sphinx | ≥ 3.0.0 | Documentation site generator (existing infrastructure; not used for this task) |
| pip | sphinx_rtd_theme | ≥ 0.4.3 | Read the Docs theme for existing Sphinx docs (not used for this task) |
| system | python | 3.11.15 | Runtime — highest explicitly documented tested version per `tox.ini` envlist `py311` |

**Note:** The new markdown document does not require any documentation build tools. It is a standalone `.md` file. The Sphinx dependencies above are listed only for completeness as they are part of the existing documentation infrastructure defined in `pyproject.toml` under `[project.optional-dependencies] docs`.

### 0.6.2 Documentation Reference Updates

No documentation link updates are required. The new file is placed in `blitzy/documentation/scapy_0925ada48540.md`, which is outside the existing documentation tree. No existing files contain links to this new document, and no existing links need to be changed.

## 0.7 Coverage and Quality Targets

### 0.7.1 Documentation Coverage Metrics

**Current coverage analysis for the user's questions:**

| Question | Existing Documentation | Coverage |
|----------|----------------------|----------|
| Q1: Interface selection | `doc/scapy/routing.rst` shows `conf.route` usage; `doc/scapy/usage.rst` mentions `send()` handles routing | ~15% — user-facing examples only, no internal code-path trace |
| Q2: MAC/ARP resolution | `doc/scapy/routing.rst` shows `getmacbyip()` one-liner example | ~10% — no neighbor resolution framework, no gateway indirection |
| Q3: ARP caching behavior | Not documented anywhere | 0% |
| Q4: No-route error handling | Not documented anywhere | 0% |

**Target coverage:** 100% — every question must be answered with complete code-path rationale, citing specific source lines.

**Coverage gaps to address:**
- `scapy/route.py:Route.route()` algorithm: Currently 0% documented at the code level, target 100%
- `scapy/layers/l2.py:getmacbyip()` full logic: Currently 10% (usage example only), target 100%
- `scapy/config.py:CacheInstance` TTL behavior: Currently 0%, target 100%
- `scapy/sendrecv.py:_interface_selection()`: Currently 0%, target 100%
- `scapy/error.py:ScapyNoDstMacException`: Currently 0%, target 100%

### 0.7.2 Documentation Quality Criteria

**Completeness requirements:**
- All four questions answered with specific code-line references
- Every claim substantiated by a source file path and line range
- All decision branches (cache hit/miss, gateway/direct, route found/not-found) documented
- Mermaid diagrams for each major flow

**Accuracy validation:**
- Code references verified against the actual source files in the repository
- Behavior claims validated by running Scapy in the Python 3.11 virtual environment
- Route table output and cache behavior confirmed by executing test scripts

**Clarity standards:**
- Progressive disclosure: start with the high-level answer, then trace the code path
- Consistent terminology: "route tuple", "neighbor resolver", "ARP cache", "route cache"
- Each section begins with a plain-English summary before diving into code analysis
- Code snippets kept to essential lines, with source citations for full context

**Maintainability:**
- Source citations in the format `Source: scapy/route.py:146-197` for traceability
- Function signatures included so future readers can locate the code
- No hard-coded line numbers without also naming the function/class

### 0.7.3 Example and Diagram Requirements

- **Minimum examples per question:** At least one Scapy interactive session example showing the behavior
- **Diagram types required:** Mermaid flowcharts for interface selection flow, ARP resolution chain, caching architecture, and no-route error path
- **Code example testing:** All examples validated by executing them in the Scapy 2026.04.09 virtual environment on Python 3.11
- **Visual content freshness:** Diagrams and examples reflect the current state of the `scapy_0925ada48540` branch

## 0.8 Scope Boundaries

### 0.8.1 Exhaustively In Scope

- **New documentation file:**
  - `blitzy/documentation/scapy_0925ada48540.md` — the primary deliverable

- **Source files to analyze and cite (read-only):**
  - `scapy/route.py` — Route class, routing algorithm, route cache
  - `scapy/route6.py` — IPv6 routing (referenced for completeness)
  - `scapy/layers/l2.py` — Neighbor class, DestMACField, getmacbyip, _arp_cache, ARP packet
  - `scapy/layers/inet.py` — IP class with route() method, inet_register_l3, DestIPField
  - `scapy/sendrecv.py` — send(), sendp(), _interface_selection(), _send(), sr(), srp()
  - `scapy/config.py` — conf singleton, CacheInstance, NetCache, raise_no_dst_mac
  - `scapy/interfaces.py` — NetworkInterfaceDict, get_working_if(), resolve_iface()
  - `scapy/error.py` — ScapyNoDstMacException, warning(), log_runtime
  - `scapy/arch/__init__.py` — Platform dispatch, get_if_hwaddr, get_if_addr
  - `scapy/arch/linux.py` — read_routes() from /proc/net/route
  - `scapy/consts.py` — Platform detection constants (LINUX, BSD, WINDOWS)
  - `scapy/data.py` — ETHER_BROADCAST and other constants

- **Existing documentation files to reference (read-only, for style and context):**
  - `doc/scapy/routing.rst` — Existing routing documentation
  - `doc/scapy/usage.rst` — Existing usage documentation
  - `doc/scapy/conf.py` — Sphinx build configuration

- **Temporary scripts for testing (created and deleted, not committed):**
  - Python scripts run in the virtual environment to validate routing and ARP behavior

- **Environment setup and validation:**
  - Python 3.11 virtual environment at `/tmp/scapy_venv`
  - Scapy installed in editable mode from the repository
  - Interactive session validation via `from scapy.all import *`

### 0.8.2 Explicitly Out of Scope

- **Source code modifications** — No changes to any `.py` file in the repository
- **Existing documentation updates** — No changes to `doc/scapy/routing.rst`, `doc/scapy/usage.rst`, or any other existing documentation file
- **Test file modifications** — No changes to files in the `test/` directory
- **Documentation build configuration changes** — No changes to `doc/scapy/conf.py`, `.readthedocs.yml`, `tox.ini`, or `pyproject.toml`
- **Feature additions or code refactoring** — Explicitly excluded by user instruction
- **IPv6 routing analysis** — While `scapy/route6.py` exists, the user's questions are IPv4-focused
- **Windows/BSD/Solaris platform specifics** — The analysis focuses on the Linux backend as the execution environment; other platform backends are noted but not traced in detail
- **TLS, contrib, or protocol-specific documentation** — Unrelated to the user's routing/ARP questions
- **Deployment configuration changes** — No CI/CD or deployment modifications

## 0.9 Execution Parameters

### 0.9.1 Documentation-Specific Instructions

- **Environment setup command:**
  ```
  python3.11 -m venv /tmp/scapy_venv && source /tmp/scapy_venv/bin/activate && pip install -e .
  ```
- **Documentation validation command:** Verify the generated markdown renders correctly and all internal links are valid
- **Interactive session command:**
  ```
  source /tmp/scapy_venv/bin/activate && python3.11 -m scapy
  ```
- **Diagram generation:** Mermaid diagrams embedded directly in the markdown file (rendered by any Mermaid-compatible viewer)
- **Default format:** Markdown (`.md`) — consistent with the `SWE-AtlasQnA-Repo` implementation rule
- **Citation requirement:** Every technical claim must reference a specific source file and line range
- **Style guide:** Follow the established Scapy documentation style from `doc/scapy/routing.rst` — use interactive session examples with `>>>` prompts, include console output, and provide plain-English explanations before code references
- **Documentation output path:** `blitzy/documentation/scapy_0925ada48540.md`

## 0.10 Rules for Documentation

The following rules are derived from the user's explicit instructions and the `SWE-AtlasQnA-Repo` implementation rule:

- **Do not modify any existing files in the source repository.** This is the paramount constraint. All analysis is read-only; the only write operation is creating the new markdown document in `blitzy/documentation/`.
- **Do not make assumptions — base all answers on the code as the truth.** Every claim must be traceable to a specific function, class, or code block in the Scapy source tree. Speculative or hypothetical behavior must not be presented as fact.
- **Provide thinking/rationale behind the answers.** Each answer must explain *why* the code behaves as described, not just *what* it does. This means tracing decision branches, explaining algorithm choices, and citing the specific code that implements each behavior.
- **Create a single markdown document named `scapy_0925ada48540.md`.** The filename matches the source branch name as required by the implementation rule.
- **Place the document in the `blitzy/documentation` directory.** This directory must be created if it does not exist.
- **Temporary scripts for testing are acceptable** but must not persist as changes to the repository. They may be run in the virtual environment to validate conclusions.
- **Leave the actual codebase unchanged.** This is the user's explicit instruction, reinforcing the implementation rule. No `.py`, `.rst`, `.yml`, or any other file in the repository may be modified.

## 0.11 References

### 0.11.1 Source Files Searched and Analyzed

The following files and folders were systematically searched across the codebase to derive all conclusions in this Agent Action Plan:

**Core routing and interface selection:**
| File | Lines Examined | Key Findings |
|------|---------------|--------------|
| `scapy/route.py` | 1-220 (complete) | `Route` class with `route()` longest-prefix-match algorithm, route cache dict, `resync()` from OS, `invalidate_cache()` |
| `scapy/interfaces.py` | 1-424 (complete) | `NetworkInterfaceDict`, `get_working_if()` selects default interface by smallest mask, `resolve_iface()` fallback to dummy |
| `scapy/sendrecv.py` | 396-700 | `send()`, `sendp()`, `_interface_selection()`, `_send()`, `sr()`, `srp()`, `srp1()` — full L3/L2 send/receive API |

**ARP and neighbor resolution:**
| File | Lines Examined | Key Findings |
|------|---------------|--------------|
| `scapy/layers/l2.py` | 1-1137 (complete) | `Neighbor` class, `DestMACField.i2h()`, `getmacbyip()` with ARP broadcast, `_arp_cache` (120s TTL), `ARP` packet class, `arping()`, `arpcachepoison()` |
| `scapy/layers/inet.py` | 503-580, 1117-1131 | `IP` class with `route()`, `DestIPField`, `inet_register_l3()` wiring Ether+IP to `getmacbyip()` |

**Configuration and caching:**
| File | Lines Examined | Key Findings |
|------|---------------|--------------|
| `scapy/config.py` | 350-530, 840-920 | `CacheInstance` with TTL-based `_timetable`, `NetCache` container, `conf.raise_no_dst_mac`, `conf.loopback_name`, `conf.netcache` |

**Error handling:**
| File | Lines Examined | Key Findings |
|------|---------------|--------------|
| `scapy/error.py` | 1-138 (complete) | `ScapyNoDstMacException`, `Scapy_Exception`, `warning()` → `log_runtime.warning()`, `ScapyFreqFilter` for warning throttling |

**Platform abstraction:**
| File | Lines Examined | Key Findings |
|------|---------------|--------------|
| `scapy/arch/__init__.py` | 1-152 (complete) | Platform dispatch (LINUX → `linux.py`, BSD → `bpf/`, Windows → `windows/`), `get_if_hwaddr()`, `get_if_addr()` |
| `scapy/arch/linux.py` | 231-315 | `read_routes()` parsing `/proc/net/route`, loopback route, flags filtering (RTF_UP, RTF_REJECT) |

**Project configuration:**
| File | Lines Examined | Key Findings |
|------|---------------|--------------|
| `pyproject.toml` | 1-end (complete) | Python ≥ 3.7 < 4, Sphinx ≥ 3.0.0 docs deps, `setuptools` build backend |
| `tox.ini` | 1-60 | Test matrix includes py{27-311}, confirming Python 3.11 as highest tested version |
| `.readthedocs.yml` | 1-end (complete) | Ubuntu 20.04, Python 3.9 for RTD builds |
| `setup.py` | (summary only) | Legacy build script with `sdist`/`build_py` hooks |

**Existing documentation (read-only reference):**
| File | Lines Examined | Key Findings |
|------|---------------|--------------|
| `doc/scapy/routing.rst` | 1-106 (complete) | Surface-level routing API docs: `conf.route`, `getmacbyip()` example, interface listing |
| `doc/scapy/usage.rst` | 242-260, 1065-1093 | Send/receive usage examples, routing table manipulation |
| `doc/scapy/conf.py` | 1-60 | Sphinx config: `needs_sphinx = '3.0.0'`, autodoc + napoleon extensions, `scapy_doc` custom extension |

**Folder structure explored:**
| Folder | Children Examined | Purpose |
|--------|-------------------|---------|
| `/` (repo root) | All 5 folders + 11 files | Project scaffold, CI, licensing, packaging |
| `scapy/` | All 33 files + 7 subfolders | Core runtime package |
| `scapy/arch/` | All 6 files + 2 subfolders | OS-specific backends |
| `scapy/layers/` | Key files (l2.py, inet.py) | Protocol layers |
| `doc/` | All 4 subfolders | Documentation tree |

### 0.11.2 Attachments

No attachments were provided for this project. There are no Figma URLs, uploaded files, or external design references.

### 0.11.3 Technical Specification Sections Consulted

The following tech spec sections were retrieved for background context:
- **1.1 Executive Summary** — Project overview, stakeholders, value proposition
- **4.1 HIGH-LEVEL SYSTEM WORKFLOW** — End-to-end processing architecture, feature dependency flow
- **4.3 NETWORK OPERATIONS WORKFLOWS** — Stimulus-response workflow, packet capture workflow
- **4.8 PLATFORM ABSTRACTION AND SOCKET SELECTION** — Platform detection, routing and interface selection, socket creation
- **4.9 ERROR HANDLING AND RECOVERY** — Error propagation architecture, recovery strategies

