# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

### 0.1.1 Core Feature Objective

Based on the prompt, the Blitzy platform understands that the new feature requirement is to produce a comprehensive exploratory documentation artifact for the Scapy packet manipulation library at commit `0925ada4`. The user is not requesting modifications to Scapy's source code but rather a hands-on investigation and a written report that captures the following discrete objectives:

- **Scapy Shell Startup**: Launch the Scapy interactive console and document the full startup banner, including the ASCII art logo, version string, loaded layers, and IPython detection behavior as implemented in `scapy/main.py` (the `interact()` function, lines 503–716).
- **Version Identification**: Report the exact Scapy version as computed by the `_version()` chain in `scapy/__init__.py`, which resolves through four fallback methods: `SCAPY_VERSION` environment variable → `scapy/VERSION` file → `git archive` substitution → `git describe --tags`.
- **ICMP Echo Request Construction**: Create an `IP(dst="127.0.0.1")/ICMP()` packet and catalog every IP header field, distinguishing user-supplied values from auto-populated defaults (e.g., `version=4`, `ttl=64`, `proto=icmp`, `id=1`) and deferred fields computed at build time (e.g., `ihl=None`, `len=None`, `chksum=None`).
- **`show()` Output**: Capture and describe the hierarchical output produced by `Packet.show()` (line 1459 of `scapy/packet.py`) and contrast it with `show2()` (line 1473), which builds the packet first to resolve computed fields.
- **Localhost ICMP Transmission**: Send the crafted ICMP echo-request to `127.0.0.1` using `sr1()` and describe the observed network-layer behavior, including socket selection, routing resolution, kernel echo-reply generation, and the `SndRcvHandler` matching engine.
- **IP Header Construction Internals**: Trace through the source code to explain how the `IP` class (line 521 of `scapy/layers/inet.py`) declares its `fields_desc`, how `SourceIPField` (line 854 of `scapy/fields.py`) auto-resolves the source address via the routing table, how `DestIPField` resolves via payload bindings, and how `post_build()` (line 539) computes `ihl`, `len`, and `chksum` from raw bytes.
- **Test Suite Execution**: Run the UTscapy test harness (`scapy/tools/UTscapy.py`) against the Linux non-root configuration (`test/configs/linux.utsc`) and report the total number of test campaigns, individual test cases, and the pass/fail summary.
- **No Source Modifications**: The user explicitly requires that no existing source files be modified; any temporary scripts created during exploration must be cleaned up afterward.

Implicit requirements detected:
- The output document must be named `scapy_0925ada48540.md` (matching the branch name) and placed in the `blitzy/documentation/` directory
- All answers must be grounded in the actual source code rather than assumptions
- Thinking and rationale must accompany each answer

### 0.1.2 Special Instructions and Constraints

- **Read-Only Repository**: The user mandates: "Don't modify any source files, if you need to create temp scripts, just clean them up after." This constrains all work to observation, execution, and documentation only.
- **Implementation Rule — SWE-AtlasQnA-Repo**: A project rule requires creating a single markdown document named `<source_branch_name>.md` in the `blitzy/documentation` directory. The document must comprehensively answer the questions posed in the prompt with thinking/rationale, be grounded in the code as truth, and must not add any other code to the source repository.
- **Specific Commit**: The repository is pinned at commit `0925ada4` ("Add AUTOSAR PDUTransport/PDU with initial batch of tests (#3933)"). The version computed by `scapy/__init__.py` is derived from the git timestamp fallback since no upstream tag exactly matches this commit.
- **Environment**: Python 3.10 is the highest explicitly documented supported runtime (per `pyproject.toml` classifiers and the GitHub Actions CI matrix). The virtual environment must include all optional dependencies (`ipython`, `pyx`, `cryptography>=2.0`, `matplotlib`) for full-fidelity exploration.

### 0.1.3 Technical Interpretation

These feature requirements translate to the following technical implementation strategy:

- To **document the Scapy shell startup**, we will execute `python -m scapy` or invoke `scapy.main.interact()` in a non-interactive capture mode, recording the banner output that includes the ASCII logo, version display, and random quote as constructed in `interact()` lines 589–655 of `scapy/main.py`.
- To **identify the version**, we will read `scapy.VERSION` at runtime (resolving through the `_version()` chain in `scapy/__init__.py`) and document each fallback method.
- To **create and inspect an ICMP packet**, we will construct `IP(dst="127.0.0.1")/ICMP()`, call `show()` and `show2()`, and document the difference between deferred fields (`None`) and computed fields (populated by `post_build()`).
- To **send a packet to localhost**, we will use `sr1()` with `L3RawSocket` (since PF_PACKET on the loopback interface may not return matched replies in container environments) and document the full network-layer exchange.
- To **explain IP header construction**, we will trace through `IP.fields_desc` (line 524), `SourceIPField.__findaddr()` (line 862 of `scapy/fields.py`), `IP.post_build()` (line 539 of `scapy/layers/inet.py`), and the `bind_layers(IP, ICMP, frag=0, proto=1)` registration at line 1112.
- To **run the test suite**, we will invoke `python -m scapy.tools.UTscapy -c ./test/configs/linux.utsc -N -K scanner` in non-root mode and capture the pass/fail summary.
- To **produce the deliverable**, we will create `blitzy/documentation/scapy_0925ada48540.md` containing all findings, structured by question with source-code references and rationale.

## 0.2 Repository Scope Discovery

### 0.2.1 Comprehensive File Analysis

The repository is the Scapy project at commit `0925ada4` on branch `scapy_0925ada48540`. Since the task is read-only exploration followed by creation of a single documentation file, the scope discovery focuses on identifying every source file that must be **read and analyzed** (not modified) to answer the user's questions, plus the single file that will be **created**.

**Files to Read and Analyze (grouped by question)**

| Question Area | Files to Analyze | Purpose |
|---|---|---|
| Shell Startup | `scapy/main.py` (lines 503–716) | `interact()` function: banner construction, IPython detection, session init |
| Shell Startup | `scapy/themes.py` | `DefaultTheme`, `apply_ipython_style` — color rendering of the banner |
| Shell Startup | `scapy/config.py` | `conf` singleton — `conf.version`, `conf.fancy_prompt`, `conf.load_layers` |
| Version | `scapy/__init__.py` (lines 21–177) | `_version()` chain: env → VERSION file → git archive → git describe → fallback |
| Version | `scapy/VERSION` (if present) | Packaged version string |
| Version | `pyproject.toml` (line 78) | `version = { attr="scapy.VERSION" }` — setuptools dynamic version |
| ICMP Packet | `scapy/layers/inet.py` (lines 521–651) | `IP` class: `fields_desc`, `post_build()`, `route()` |
| ICMP Packet | `scapy/layers/inet.py` (lines 952–1011) | `ICMP` class: `fields_desc`, `post_build()`, type/code enums |
| ICMP Packet | `scapy/layers/inet.py` (line 1112) | `bind_layers(IP, ICMP, frag=0, proto=1)` |
| show() Output | `scapy/packet.py` (lines 1383–1486) | `_show_or_dump()`, `show()`, `show2()` |
| IP Header Construction | `scapy/fields.py` (lines 729–755, 796–890) | `DestField`, `IPField`, `SourceIPField` — auto-resolution logic |
| IP Header Construction | `scapy/packet.py` (lines 141–206, 746–806) | `Packet.__init__()`, `build()`, `do_build()`, `post_build()` |
| Send/Receive | `scapy/sendrecv.py` (lines 375–448, 635–700) | `_send()`, `send()`, `sr()`, `sr1()` |
| Send/Receive | `scapy/supersocket.py` | `SuperSocket`, `L3RawSocket` |
| Send/Receive | `scapy/route.py` | IPv4 routing table, `route()` method |
| Test Suite | `scapy/tools/UTscapy.py` | Test runner: campaign parsing, execution, result reporting |
| Test Suite | `test/configs/linux.utsc` | Linux test configuration: file patterns, preexec, keyword exclusions |
| Test Suite | `test/**/*.uts` (190 files) | Individual test scripts (6025 tests across 168 campaigns) |

**Integration point discovery:**
- `scapy/all.py` — The umbrella import surface that re-exports symbols from 30+ internal modules; entry point for understanding what is loaded in the interactive namespace
- `scapy/layers/all.py` — Dynamic layer loader that reads `conf.load_layers` and imports each protocol module
- `scapy/arch/linux.py` — Linux-specific socket backends (`L3PacketSocket`, `L2Socket`) used during `send()`/`sr1()`
- `scapy/arch/__init__.py` — Facade that conditionally imports the active platform backend
- `scapy/data.py` — Protocol number tables (`IP_PROTOS`, `TCP_SERVICES`) used by field enums
- `scapy/consts.py` — Platform detection flags (`LINUX`, `WINDOWS`, etc.)

**Repository structure summary:**

```
scapy/                   # Core runtime package (34 modules + 7 subpackages)
├── __init__.py          # Version computation
├── __main__.py          # Module execution entry point
├── all.py               # Umbrella import surface
├── main.py              # Interactive console startup
├── packet.py            # Packet base class, build/dissect/show
├── fields.py            # 60+ field types (IPField, SourceIPField, etc.)
├── config.py            # Global conf singleton
├── sendrecv.py          # send(), sr(), sr1(), sniff()
├── layers/
│   ├── inet.py          # IP, TCP, UDP, ICMP definitions
│   └── ...              # 48+ built-in protocol layers
├── tools/
│   └── UTscapy.py       # Test runner
├── arch/
│   ├── linux.py         # Linux PF_PACKET sockets
│   └── ...
└── ...
test/                    # Test infrastructure
├── configs/
│   └── linux.utsc       # Linux test configuration
├── regression.uts       # Core regression tests
├── scapy/layers/
│   └── inet.uts         # IP/ICMP layer tests
└── ...                  # 190 .uts test files total
```

### 0.2.2 Web Search Research Conducted

No external web searches were necessary for this task. All questions can be fully answered from the source code in the repository. The Scapy project is self-contained with comprehensive inline documentation, and the user explicitly requests code-grounded analysis rather than general best-practice research.

### 0.2.3 New File Requirements

A single new file will be created as the deliverable:

- **CREATE**: `blitzy/documentation/scapy_0925ada48540.md` — A comprehensive markdown document answering all of the user's questions about Scapy shell startup, version, ICMP packet construction, `show()` output, localhost packet transmission, IP header internals, and test suite results. This file will include:
  - Full Scapy shell startup banner reproduction
  - Version identification with source-code tracing
  - Annotated `show()` and `show2()` output for `IP()/ICMP()`
  - Network-layer analysis of ICMP echo to localhost
  - Source-code walkthrough of IP header construction
  - UTscapy test suite summary with pass/fail counts
  - Thinking and rationale grounding every answer in code

No new test files, configuration files, or source modules are required.

## 0.3 Dependency Inventory

### 0.3.1 Private and Public Packages

All packages are sourced from the public PyPI registry. The versions below are taken directly from the `pyproject.toml` manifest and the `tox.ini` test dependency list at commit `0925ada4`, verified against the installed environment.

| Registry | Package | Version | Purpose |
|---|---|---|---|
| PyPI | `scapy` | 2026.04.13 (editable install from commit `0925ada4`) | Core packet manipulation library under investigation |
| PyPI | `setuptools` | ≥62.0.0 (build-system requires in `pyproject.toml`) | Build backend for the Scapy package |
| PyPI | `ipython` | 8.39.0 (installed; optional `cli` extra) | Enhanced interactive shell used by `scapy/main.py` `interact()` |
| PyPI | `cryptography` | ≥2.0 (per `pyproject.toml`); 46.0.7 installed | TLS/SSL cryptographic operations in `scapy/layers/tls/` and `scapy/layers/ipsec.py` |
| PyPI | `pyx` | 0.17 (installed; optional `all` extra) | PostScript/PDF packet diagram rendering in `scapy/packet.py` canvas_dump |
| PyPI | `matplotlib` | 3.10.8 (installed; optional `all` extra) | Graphical packet visualization in `scapy/libs/matplot.py` |
| PyPI | `mock` | 5.2.0 (test dependency from `tox.ini`) | Test mocking framework used in UTscapy test scripts |
| PyPI | `coverage` | 7.13.5 (test dependency from `tox.ini`) | Code coverage measurement during test execution |
| PyPI | `python-can` | 4.6.1 (test dependency from `tox.ini`) | CAN bus support for automotive protocol testing in `scapy/contrib/automotive/` |

**Runtime Python Version:**

| Runtime | Required Range | Highest Classified Version | Installed Version |
|---|---|---|---|
| Python | ≥3.7, <4 (`pyproject.toml` line 17) | 3.10 (`pyproject.toml` classifiers + CI matrix) | 3.10.20 |

The highest explicitly documented supported Python version is **3.10**, based on `pyproject.toml` classifiers (lines 30–31: `Programming Language :: Python :: 3.10`) and the GitHub Actions CI matrix in `.github/workflows/unittests.yml` (line 72: `python: ["3.7", "3.8", "3.9", "3.10"]`). The `tox.ini` mentions `py311` in its envlist, but the CI does not test 3.11 for the main unit test suite, and the classifiers do not include it.

### 0.3.2 Dependency Updates

No dependency updates are required for this task. The deliverable is a standalone markdown document that does not introduce new imports, modify existing code, or alter the project's dependency graph. The environment was set up solely to enable live Scapy execution for observation.

**Import state**: No files require import changes since no source files are being modified.

**External reference state**: No configuration files, build files, or CI definitions require updates.

## 0.4 Integration Analysis

### 0.4.1 Existing Code Touchpoints

Since this task creates a new documentation file without modifying any source code, the integration analysis focuses on identifying the **read-only touchpoints** — the code paths that must be traced and understood to produce accurate documentation.

**Direct code paths traced for each user question:**

- **Shell Startup Path**:
  - `scapy/__main__.py` → `scapy/main.py:interact()` (line 503) → `init_session()` (line 415) → `_scapy_builtins()` (line 302) → `scapy/all.py` (full import surface)
  - `interact()` reads `STARTUP_FILE` / `PRESTART_FILE` via `_read_config_file()` (line 73)
  - Banner construction at lines 589–655 uses `conf.version`, `conf.color_theme`, `QUOTES` list, and `_prepare_quote()`
  - IPython detection at lines 567–587 attempts `import IPython` and falls back to `code.interact()`

- **Version Resolution Chain**:
  - `scapy/__init__.py:_version()` (line 122) → Method 0: `os.environ['SCAPY_VERSION']` → Method 1: `scapy/VERSION` file → Method 2: `_version_from_git_archive()` (line 38) → Method 3: `_version_from_git_describe()` (line 72) → Fallback: `__file__` mtime
  - At this commit, Method 3 (`git describe`) fails because no upstream tag is directly reachable, so the fallback to `__file__` modification timestamp produces a date-formatted version like `2026.04.13`

- **ICMP Packet Construction Path**:
  - `IP(dst="127.0.0.1")` → `Packet.__init__()` (line 141 of `scapy/packet.py`) → `init_fields()` populates `default_fields` from `fields_desc` → User-provided `dst` is stored in `self.fields`
  - `/ICMP()` → `Packet.__truediv__()` (line 596) → `add_payload()` (line 360) → triggers `overload_fields` via `bind_layers(IP, ICMP, frag=0, proto=1)` at line 1112 of `scapy/layers/inet.py`
  - `SourceIPField.__findaddr()` (line 862 of `scapy/fields.py`) → `conf.route.route("127.0.0.1")` → returns `("lo", "127.0.0.1", "0.0.0.0")` → source IP resolved to `127.0.0.1`

- **Build and show() Path**:
  - `show()` → `_show_or_dump()` (line 1383) iterates `fields_desc`, calls `f.i2repr()` for display — deferred fields show as `None`
  - `show2()` → `self.__class__(raw(self)).show()` (line 1486) — first calls `build()` which triggers `post_build()`, then dissects the result to show computed values
  - `IP.post_build()` (line 539): computes `ihl` from header length, `len` from total packet length, `chksum` from IP header checksum

- **Send/Receive Path**:
  - `sr1(pkt)` → `sr()` (line 635 of `scapy/sendrecv.py`) → `SndRcvHandler.sndrcv()` (line 100) → spawns send thread + `AsyncSniffer`
  - `send()` → `_send()` (line 396) → `_func(iface)` resolves to `iface.l3socket()` → `L3PacketSocket` on Linux
  - Loopback routing: `conf.route.route("127.0.0.1")` → `("lo", "127.0.0.1", "0.0.0.0")` — packets go through the `lo` interface
  - Response matching: `SndRcvHandler._process_packet()` computes `hashret()` on received packet and checks `answers()` for semantic match

- **Test Suite Path**:
  - `scapy/tools/UTscapy.py` main entry point → parses `-c test/configs/linux.utsc` → resolves 190 `.uts` test files → executes each campaign → reports pass/fail per test

**Database/Schema updates**: None — Scapy has no database.

**Dependency injections**: None — No new services or modules are being registered.

**Output integration point**: The only integration is the creation of `blitzy/documentation/scapy_0925ada48540.md` in the destination repository, which is a pure additive operation with zero coupling to existing code.

## 0.5 Technical Implementation

### 0.5.1 File-by-File Execution Plan

The implementation consists of a single file creation with no modifications to existing code.

**Group 1 — Deliverable Document:**

- **CREATE**: `blitzy/documentation/scapy_0925ada48540.md`
  - Comprehensive Q&A markdown document answering all user questions
  - Structured with sections for each question, including rationale and source-code references
  - Must include: shell startup output, version analysis, ICMP packet construction walkthrough, `show()` vs `show2()` output, localhost send/receive analysis, IP header internals explanation, and test suite summary

**Group 2 — Temporary Exploration Scripts (created and cleaned up):**

- **CREATE then DELETE**: Temporary Python scripts for non-interactive Scapy execution (e.g., packet creation, `show()` capture, `sr1()` testing) — all to be removed after their output is captured for the document

**Group 3 — Files Read for Analysis (no modifications):**

| File | Lines of Interest | What to Extract |
|---|---|---|
| `scapy/main.py` | 49–59, 503–716 | QUOTES list, `interact()` banner logic, IPython detection |
| `scapy/__init__.py` | 21–177 | `_version()` method chain with four fallbacks |
| `scapy/layers/inet.py` | 521–551, 952–1011, 1112 | `IP.fields_desc`, `IP.post_build()`, `ICMP` class, `bind_layers` |
| `scapy/fields.py` | 729–755, 796–890 | `DestField`, `DestIPField`, `IPField`, `SourceIPField` |
| `scapy/packet.py` | 141–206, 746–806, 1383–1486 | `__init__()`, `build()`, `_show_or_dump()`, `show()`, `show2()` |
| `scapy/sendrecv.py` | 375–448, 635–700 | `send()`, `_send()`, `sr()`, `sr1()` |
| `scapy/route.py` | routing resolution | `conf.route.route()` for source IP and interface selection |
| `scapy/config.py` | 848–897 | `conf.load_layers` list (48 default layers) |
| `scapy/all.py` | 1–54 | Umbrella import surface |
| `scapy/tools/UTscapy.py` | full file | Test runner architecture |
| `test/configs/linux.utsc` | full file | Test configuration (190 files, keyword exclusions) |

### 0.5.2 Implementation Approach per File

**Step 1 — Environment Setup and Verification:**
- Install Python 3.10 (highest explicitly classified version)
- Create virtualenv and install Scapy in editable mode with `[all]` extras
- Install test dependencies: `mock`, `coverage`, `python-can`
- Verify: `python -c "import scapy; print(scapy.VERSION)"` → confirms `2026.04.13`

**Step 2 — Shell Startup Capture:**
- Execute `scapy.main.interact()` in a capture context to record the banner
- Document the ASCII logo (19 lines), version line (`Version %s` % `conf.version`), welcome message, and random quote selection
- Note: IPython is used when available (line 571 of `scapy/main.py`); otherwise falls back to `code.interact()`

**Step 3 — ICMP Packet Construction and Inspection:**
- Create `IP(dst="127.0.0.1")/ICMP()` and capture `show()` output
- Document each IP field: `version=4` (default), `ihl=None` (deferred), `tos=0x0`, `len=None` (deferred), `id=1`, `flags=`, `frag=0`, `ttl=64`, `proto=icmp` (auto-set by `bind_layers`), `chksum=None` (deferred), `src=127.0.0.1` (auto-resolved via `SourceIPField`), `dst=127.0.0.1` (user-specified)
- Capture `show2()` output to show resolved fields: `ihl=5`, `len=28`, `chksum=0x7cde`

**Step 4 — Localhost ICMP Transmission:**
- Use `sr1(IP(dst="127.0.0.1")/ICMP(), timeout=3)` with `L3RawSocket` for reliable loopback operation
- Document: packet sent, kernel generates echo-reply (type=0), response matched by `SndRcvHandler` via `hashret()`/`answers()`, response shows `type=echo-reply`, `chksum=0x0` (kernel-computed)

**Step 5 — Source Code Walkthrough:**
- Trace `IP.fields_desc` declarations at line 524 of `scapy/layers/inet.py`
- Explain `SourceIPField.__findaddr()` at line 862 of `scapy/fields.py` — resolves source IP by routing to the destination
- Explain `IP.post_build()` at line 539 — computes `ihl` (header length / 4), `len` (total packet length), and `chksum` (IP header checksum)
- Explain `bind_layers(IP, ICMP, frag=0, proto=1)` — sets `proto=1` automatically when ICMP is the payload

**Step 6 — Test Suite Execution and Summary:**
- Run `python -m scapy.tools.UTscapy -c ./test/configs/linux.utsc -N -K scanner`
- Capture total test count: 190 test files, 168 campaigns, approximately 6025 individual tests
- Document pass/fail summary from the UTscapy output format

**Step 7 — Document Assembly:**
- Compile all findings into `blitzy/documentation/scapy_0925ada48540.md`
- Structure with clear headings per question, code blocks for output, and rationale sections
- Clean up any temporary scripts

## 0.6 Scope Boundaries

### 0.6.1 Exhaustively In Scope

**Files to CREATE:**
- `blitzy/documentation/scapy_0925ada48540.md` — The sole deliverable

**Files to READ and ANALYZE (no modifications):**
- `scapy/__init__.py` — Version computation chain
- `scapy/__main__.py` — Module execution entry
- `scapy/main.py` — Interactive console startup, banner, `interact()`
- `scapy/all.py` — Umbrella import surface
- `scapy/packet.py` — `Packet` base class, `build()`, `show()`, `show2()`, `_show_or_dump()`
- `scapy/fields.py` — `IPField`, `SourceIPField`, `DestField`, `DestIPField` and 60+ field types
- `scapy/layers/inet.py` — `IP`, `ICMP`, `TCP`, `UDP`, `bind_layers`, `post_build()`
- `scapy/layers/all.py` — Layer loading mechanism
- `scapy/sendrecv.py` — `send()`, `sr()`, `sr1()`, `SndRcvHandler`
- `scapy/supersocket.py` — `SuperSocket`, `L3RawSocket`
- `scapy/route.py` — IPv4 routing table and `route()` method
- `scapy/config.py` — `conf` singleton, `load_layers` list, socket backend assignments
- `scapy/consts.py` — Platform detection flags
- `scapy/data.py` — Protocol number tables (`IP_PROTOS`)
- `scapy/themes.py` — Color themes for banner rendering
- `scapy/volatile.py` — `RandShort`, `RandInt` — random field generators
- `scapy/compat.py` — `raw()` function for packet serialization
- `scapy/error.py` — Logging infrastructure
- `scapy/interfaces.py` — Interface discovery
- `scapy/arch/__init__.py` — Platform backend facade
- `scapy/arch/linux.py` — Linux `L3PacketSocket` implementation
- `scapy/tools/UTscapy.py` — Test runner harness
- `test/configs/linux.utsc` — Linux test configuration
- `test/**/*.uts` — 190 test script files (for counting and representative execution)
- `pyproject.toml` — Package metadata, Python version requirements, entry points
- `setup.py` — Legacy build script, VERSION file generation
- `tox.ini` — Test matrix, QA environments
- `.github/workflows/unittests.yml` — CI matrix (Python versions, test environments)
- `README.md` — Project overview, quick-start examples

### 0.6.2 Explicitly Out of Scope

- **Source code modifications** to any existing Scapy file — the user explicitly prohibits this
- **New Python modules** or scripts in the `scapy/` or `test/` directories
- **Dependency version upgrades** or additions to `pyproject.toml` or `tox.ini`
- **CI/CD pipeline changes** to `.github/workflows/` or `.travis.yml`
- **Performance optimization** of Scapy's packet engine
- **Refactoring** of any existing code
- **Windows, macOS, or BSD** platform-specific analysis — the user's context is Linux-based
- **Protocol layer additions** or modifications to `scapy/layers/` or `scapy/contrib/`
- **Documentation changes** to `doc/` (the Sphinx-based manual) — the deliverable goes to `blitzy/documentation/`
- **TLS/SSL subsystem analysis** beyond what is needed for the test suite summary
- **Automotive protocol analysis** beyond noting the AUTOSAR PDU addition in the current commit

## 0.7 Rules for Feature Addition

The following rules are explicitly emphasized by the user and must be strictly observed:

- **No Source File Modifications**: The user states: "Don't modify any source files." No file in the `scapy/`, `test/`, `doc/`, or `.config/` directories may be altered. The only writable target is the `blitzy/documentation/` directory.

- **Temporary Script Cleanup**: The user states: "if you need to create temp scripts, just clean them up after." Any helper scripts created for non-interactive Scapy execution must be deleted once their output is captured. The final repository state must contain only the original Scapy code plus the deliverable document.

- **SWE-AtlasQnA-Repo Implementation Rule**: The project rule mandates:
  - Create a new markdown document named `scapy_0925ada48540.md` (matching the source branch name)
  - Provide thinking and rationale behind all answers
  - Base all answers on the code as the truth — no assumptions
  - Do not modify any existing files in the source repository
  - Do not add any other code in the source repository besides the requested document
  - Place the generated document in the `blitzy/documentation` directory

- **Code-Grounded Analysis**: Every claim about Scapy's behavior must reference specific file paths and line numbers from the repository. Statements like "Scapy typically does X" are insufficient; the correct form is "At line 539 of `scapy/layers/inet.py`, `IP.post_build()` computes the checksum by calling `checksum(p)` on the header bytes."

- **Commit-Specific Analysis**: All analysis must be performed against commit `0925ada4` specifically. The version number, test counts, and code references must reflect this exact commit state, not any upstream release.

## 0.8 References

### 0.8.1 Files and Folders Searched

The following files and folders were searched and analyzed across the codebase to derive the conclusions in this Agent Action Plan:

**Root-level files:**
- `pyproject.toml` — Package metadata, Python version classifiers, build system, optional dependencies
- `setup.py` — Legacy build script, `SDist`/`BuildPy` hooks for VERSION file
- `tox.ini` — Full test matrix (py37–py311), QA environments (flake8, mypy, docs, twine)
- `.readthedocs.yml` — Documentation build config (Python 3.9, Ubuntu 20.04)
- `.travis.yml` — Travis CI pipeline (Python 3.8 isotp test)
- `.appveyor.yml` — Windows CI pipeline (Python 3.7, Npcap)
- `README.md` — Project overview, supported platforms, quick-start
- `CONTRIBUTING.md` — Contributor guide
- `LICENSE` — GPL-2.0-only terms
- `.gitattributes` — Export-subst for `scapy/__init__.py`
- `run_scapy.bat` — Windows launcher

**`scapy/` package (core runtime):**
- `scapy/__init__.py` — Version computation chain (4 fallback methods)
- `scapy/__main__.py` — Module execution entry
- `scapy/main.py` — Interactive console (`interact()`, session persistence, QUOTES)
- `scapy/all.py` — Umbrella import (re-exports from 30+ modules)
- `scapy/packet.py` — `Packet` class (build/dissect/show lifecycle)
- `scapy/fields.py` — 60+ field types (IPField, SourceIPField, DestField)
- `scapy/layers/inet.py` — IP, TCP, UDP, ICMP, bind_layers
- `scapy/layers/all.py` — Dynamic layer loader
- `scapy/layers/__init__.py` — Layers package marker
- `scapy/config.py` — `conf` singleton
- `scapy/consts.py` — Platform detection
- `scapy/sendrecv.py` — send/sr/sr1/sniff
- `scapy/supersocket.py` — Socket abstractions
- `scapy/route.py` — IPv4 routing
- `scapy/themes.py` — Color themes
- `scapy/data.py` — Protocol tables
- `scapy/volatile.py` — Random value generators
- `scapy/compat.py` — Compatibility utilities
- `scapy/error.py` — Logging
- `scapy/interfaces.py` — Interface management

**`scapy/tools/` package:**
- `scapy/tools/UTscapy.py` — Test runner harness
- `scapy/tools/__init__.py` — Package marker

**`scapy/arch/` package:**
- `scapy/arch/__init__.py` — Platform backend facade
- `scapy/arch/linux.py` — Linux PF_PACKET sockets

**`.github/` folder:**
- `.github/workflows/unittests.yml` — GitHub Actions CI (Python 3.7–3.10 matrix)

**`test/` folder:**
- `test/__init__.py` — Package marker
- `test/configs/linux.utsc` — Linux test configuration
- `test/configs/README.md` — Test config documentation
- `test/testsocket.py` — Reusable test socket helpers
- `test/**/*.uts` — 190 individual test script files (6025 tests, 168 campaigns)

### 0.8.2 Attachments

No attachments were provided by the user for this project.

### 0.8.3 External References

- **Repository**: `github.com/secdev/scapy` at commit `0925ada48540` (branch `scapy_0925ada48540`)
- **Commit Message**: "Add AUTOSAR PDUTransport/PDU with initial batch of tests (#3933)"
- **Documentation**: `scapy.readthedocs.io` (referenced in `pyproject.toml` project.urls)
- **PyPI Package**: `scapy` — GPL-2.0-only, Python ≥3.7 <4

### 0.8.4 Key Findings Summary

| Finding | Evidence |
|---|---|
| Scapy version at this commit resolves to `2026.04.13` via date-based fallback | `scapy/__init__.py` line 160 — `d.strftime('%Y.%m.%d')` from `__file__` mtime |
| IP header has 13 fields in `fields_desc`, 3 deferred (`ihl`, `len`, `chksum`) | `scapy/layers/inet.py` lines 524–537 |
| Source IP auto-resolved via `SourceIPField.__findaddr()` using routing table | `scapy/fields.py` lines 862–877 |
| `show()` displays pre-build state (deferred = `None`); `show2()` builds first | `scapy/packet.py` lines 1459–1486 |
| `bind_layers(IP, ICMP, frag=0, proto=1)` at line 1112 sets proto automatically | `scapy/layers/inet.py` line 1112 |
| `IP.post_build()` computes `ihl`, `len`, `chksum` from raw bytes | `scapy/layers/inet.py` lines 539–551 |
| ICMP `post_build()` computes checksum similarly | `scapy/layers/inet.py` lines 979–984 |
| Test suite: 190 files, 168 campaigns, ~6025 individual tests | `test/configs/linux.utsc` + file scanning |
| `inet.uts` tests: 54 passed, 0 failed | UTscapy execution output |
| `regression.uts` tests: 270 passed, 16 failed (non-root environment) | UTscapy execution output |
| `L3RawSocket` required on loopback for reliable `sr1()` in containers | Empirical testing — `L3PacketSocket` yields 0 answers on `lo` |

