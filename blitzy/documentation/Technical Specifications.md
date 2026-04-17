# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

### 0.1.1 Core Feature Objective

Based on the prompt, the Blitzy platform understands that the new feature requirement is to **produce a comprehensive exploratory documentation artifact** for the Scapy packet manipulation library at commit `0925ada485406684174d6f068dbd85c4154657b3`. The user wants a markdown document named `scapy_0925ada48540.md` placed in `blitzy/documentation/` that answers all posed questions through hands-on execution and source code analysis. No source files in the repository may be modified.

The specific feature requirements are:

- **Scapy Shell Startup Analysis** — Launch the Scapy interactive console and capture/describe the full startup sequence, including the ASCII art logo, version banner, random quote, and IPython integration messaging as rendered by `scapy/main.py:interact()`.
- **Version Identification** — Determine and report the exact Scapy version produced by the `_version()` chain in `scapy/__init__.py` for this specific commit (which lacks git tags in a shallow clone, falling back to the date-based version format).
- **ICMP Echo Request Construction** — Create a basic `IP()/ICMP()` packet and demonstrate which IP header fields are auto-populated by Scapy's default value and `post_build()` mechanisms versus which fields are left as `None` for deferred computation.
- **`show()` Output Documentation** — Capture and describe the output of calling `show()` (displaying user-set and default values) and `show2()` (displaying the fully assembled packet with computed checksums, lengths, and IHL) on the crafted ICMP packet.
- **Localhost Packet Transmission** — Send a crafted ICMP echo-request to `127.0.0.1` using Scapy's `sr1()`/`send()` functions and describe the observable network-layer behavior, including the Linux PF_PACKET socket behavior on the loopback interface and kernel ICMP echo-reply generation.
- **IP Header Construction Internals** — Trace through the Scapy source code (`scapy/packet.py`, `scapy/fields.py`, `scapy/layers/inet.py`) to explain the programmatic mechanism by which the IP header is constructed: field descriptors, default values, `bind_layers()` overload fields, `post_build()` auto-computation of `ihl`, `len`, and `chksum`.
- **Test Suite Execution** — Run the UTscapy test framework using the Linux non-root configuration (`test/configs/linux.utsc`) and report the total number of tests, campaigns executed, pass/fail summary, and the nature of any failures.
- **Cleanup Obligation** — Any temporary scripts created for exploration must be removed after use; no artifacts besides the final markdown document may persist.

Implicit requirements detected:

- The document must include actual captured output (not hypothetical examples) from running Scapy in the container environment.
- The analysis must trace code paths through multiple source files to explain the IP header construction mechanism, requiring deep source code reading.
- The test suite must be run in non-root mode (`-N` flag) since the container environment does not provide the root-level network privileges required for certain tests.
- The version reported by Scapy at this commit will be date-based (e.g., `2026.04.16`) rather than a semantic version, because the shallow git clone has no tags and the `_version()` fallback chain in `scapy/__init__.py` resolves to the file modification timestamp.

### 0.1.2 Special Instructions and Constraints

- **No Source Modification Rule**: The user explicitly states: *"Don't modify any source files"*. This means all existing files under `scapy/`, `test/`, `doc/`, and root configuration files must remain untouched. Only the output markdown document is to be created.
- **Temporary Script Cleanup**: The user instructs: *"if you need to create temp scripts, just clean them up after"*. Any Python helper scripts used for exploration must be deleted after their output is captured.
- **Output Document Location**: Per the implementation rule `SWE-AtlasQnA-Repo`, the document must be placed at `blitzy/documentation/scapy_0925ada48540.md` in the destination repository.
- **Repository Integrity**: The implementation rule states: *"Do not add any other code in the source repository (besides the above requested document)"*.
- **Evidence-Based Answers**: Per the rule: *"Do not make assumptions, base your answers on the code as the truth"*. All answers must reference actual source code and observed runtime behavior.
- **Rationale Requirement**: The rule states: *"Provide thinking / rationale behind the answers"*, meaning the document must explain the *why* behind observed behaviors, not just report outputs.

### 0.1.3 Technical Interpretation

These feature requirements translate to the following technical implementation strategy:

- To **document the Scapy shell startup**, we will analyze `scapy/main.py` (lines 503–665) where the `interact()` function constructs the ASCII banner, detects terminal width, selects between full and mini logos, appends a random quote from the `QUOTES` list, and launches either an IPython or standard Python REPL.
- To **identify the version**, we will trace the `_version()` function in `scapy/__init__.py` (lines 124–168) through its four fallback methods: `SCAPY_VERSION` env var → `VERSION` file → `_version_from_git_archive()` → `_version_from_git_describe()` → file modification timestamp fallback.
- To **demonstrate ICMP echo-request construction**, we will execute `IP()/ICMP()` and capture both `show()` and `show2()` outputs, explaining the difference between deferred fields (`None` for `ihl`, `len`, `chksum`) and preset defaults (`version=4`, `ttl=64`, `id=1`, `proto=icmp` via overload_fields).
- To **analyze the IP header construction mechanism**, we will trace the code path through `Packet.__div__()` (packet.py line 596) → `add_payload()` (line 360) → `overload_fields` injection → `self_build()` (line 678) → `post_build()` (inet.py line 541) which computes `ihl`, `len`, and `chksum`.
- To **document localhost packet transmission**, we will execute `send()` and `sr1()` against `127.0.0.1`, capture sniffed traffic on the `lo` interface, and explain why the default `L3PacketSocket` (PF_PACKET) may not match the kernel's ICMP echo-reply on loopback, while `L3RawSocket` successfully captures the response.
- To **run the test suite**, we will execute `python3 -m scapy.tools.UTscapy -c ./test/configs/linux.utsc -N -f text` and parse the campaign-by-campaign results to report the aggregate pass/fail counts across all test files.

## 0.2 Repository Scope Discovery

### 0.2.1 Comprehensive File Analysis

The Scapy repository at commit `0925ada4` is a mature Python project with over 250 Python source files, 193 unit test scripts (`.uts` format), and a comprehensive CI/build configuration. The following files and directories are directly relevant to answering the user's questions.

**Core Source Files for IP Header Construction Analysis**

| File Path | Lines | Relevance |
|---|---|---|
| `scapy/packet.py` | 2,553 | `Packet` base class: `__init__()`, `__div__()`, `add_payload()`, `self_build()`, `do_build()`, `post_build()`, `show()`, `show2()`, `getfieldval()`, `init_fields()`, `bind_layers()` |
| `scapy/fields.py` | 3,868 | Field type system: `SourceIPField` (line 854), `IPField`, `BitField`, `ShortField`, `ByteField`, `XShortField`, `FlagsField`, `ByteEnumField` |
| `scapy/layers/inet.py` | 2,192 | `IP` class (line 521), `ICMP` class (line 952), `bind_layers(IP, ICMP, frag=0, proto=1)` (line 1112), IP `post_build()` (line 541) |
| `scapy/__init__.py` | ~170 | Version resolution: `_version()`, `_version_from_git_describe()`, `_version_from_git_archive()`, `VERSION` constant |
| `scapy/config.py` | 979 | `Conf` class: `conf.version`, `conf.iface`, `conf.L3socket`, `conf.route`, `conf.load_layers` |

**Interactive Console and Startup Files**

| File Path | Relevance |
|---|---|
| `scapy/main.py` | `interact()` function (line 503): banner generation, logo rendering, session initialization, IPython detection, startup file loading |
| `scapy/themes.py` | `DefaultTheme`, color rendering for banner text |
| `scapy/utils.py` | `get_terminal_width()` used for banner sizing |
| `scapy/all.py` | Convenience import module: `from scapy.all import *` loads all default layers and commands |

**Network Operations Files (Packet Send/Receive)**

| File Path | Relevance |
|---|---|
| `scapy/sendrecv.py` | `send()` (line 422), `sr1()` (line 656), `sr()` (line 635), `sniff()` (line 1308), `SndRcvHandler` (line 100) |
| `scapy/supersocket.py` | `SuperSocket` base class, socket abstraction layer |
| `scapy/arch/linux.py` | `L3PacketSocket` (PF_PACKET), Linux-specific raw socket implementation |
| `scapy/route.py` | `Route` class for IPv4 routing table lookup, used by `SourceIPField.__findaddr()` |

**Test Framework and Configuration**

| File Path | Relevance |
|---|---|
| `scapy/tools/UTscapy.py` | Test runner engine: campaign parsing, execution, output formatting |
| `test/configs/linux.utsc` | Linux non-root test configuration: lists 12+ test file globs, keyword exclusions (`osx`, `windows`, `ipv6`), `breakfailed: true` |
| `test/*.uts` | Core test campaigns (17 files): `regression.uts`, `fields.uts`, `cert.uts`, `pipetool.uts`, etc. |
| `test/scapy/layers/*.uts` | Layer-specific tests (40+ files): `inet.uts`, `dns.uts`, `l2.uts`, etc. |
| `test/contrib/*.uts` | Contrib module tests (100+ files) |
| `test/tools/*.uts` | Tool-specific tests (3 files) |

**Build and Configuration Files**

| File Path | Relevance |
|---|---|
| `pyproject.toml` | Project metadata, Python version constraint (`>=3.7, <4`), entry points, optional dependencies |
| `setup.py` | Build configuration, version injection via `_build_version()` |
| `tox.ini` | Test matrix: `py{37,38,39,310,311}`, test commands, CI environments |
| `.github/` | CI workflow definitions |
| `.travis.yml` | Legacy CI configuration |
| `.appveyor.yml` | Windows CI configuration |

**Integration Point Discovery**

- **Packet Build Pipeline**: `Packet.__div__()` → `add_payload()` → `overload_fields` injection → `self_build()` → field-by-field serialization via `addfield()` → `post_build()` for checksums/lengths → `build()` final assembly.
- **Routing Integration**: `IP.route()` (inet.py line 557) → `conf.route.route(dst)` → returns `(iface, src_ip, gateway)` tuple used by `SourceIPField.__findaddr()` to auto-resolve source IP.
- **Layer Binding**: `bind_layers(IP, ICMP, frag=0, proto=1)` (inet.py line 1112) → `bind_top_down()` (packet.py line 1953) stores `{IP: {'frag': 0, 'proto': 1}}` in `ICMP._overload_fields` → injected into `IP.overloaded_fields` when ICMP becomes IP's payload.
- **Socket Selection**: `conf.L3socket` defaults to `L3PacketSocket` on Linux → uses PF_PACKET raw sockets → has known behavior differences on the loopback interface compared to `L3RawSocket`.

### 0.2.2 New File Requirements

Only one new file is required:

- **CREATE**: `blitzy/documentation/scapy_0925ada48540.md` — The comprehensive QnA markdown document answering all user questions with captured outputs, source code explanations, and test results. This document must contain:
  - Scapy shell startup description and banner output
  - Version identification and explanation of the version resolution mechanism
  - ICMP echo-request `show()` and `show2()` output with field-by-field explanation
  - Localhost packet transmission analysis with network-layer observations
  - Detailed source code walkthrough of IP header construction
  - Full test suite pass/fail summary with campaign breakdown
  - Rationale and thinking behind each answer

No other new files are required. No existing files will be modified.

## 0.3 Dependency Inventory

### 0.3.1 Private and Public Packages

The following packages are relevant to the Scapy installation and the exploration tasks required by this feature.

**Core Runtime Dependencies**

| Registry | Package Name | Version | Purpose |
|---|---|---|---|
| PyPI | `scapy` | 0925ada4 (commit) | The project itself, installed in editable mode (`pip install -e .`) |
| PyPI | `setuptools` | ≥62.0.0 | Build system requirement per `pyproject.toml` |

Scapy's `pyproject.toml` declares **no mandatory runtime dependencies** — the core library is pure Python and relies solely on the Python standard library.

**Optional Dependencies (from `pyproject.toml` `[project.optional-dependencies]`)**

| Registry | Package Name | Version Constraint | Purpose | Install Group |
|---|---|---|---|---|
| PyPI | `ipython` | (any) | Enhanced interactive shell with tab completion and history | `cli`, `all` |
| PyPI | `pyx` | (any) | PostScript/PDF packet diagram generation | `all` |
| PyPI | `cryptography` | ≥2.0 | TLS/SSL layer support, gated by `conf.crypto_valid` | `all` |
| PyPI | `matplotlib` | (any) | Traffic plotting and visualization | `all` |

**Test Dependencies (from `tox.ini` `[testenv]` `deps`)**

| Registry | Package Name | Version Constraint | Purpose |
|---|---|---|---|
| PyPI | `mock` | (any) | Test mocking framework used in `test/answering_machines.uts` |
| PyPI | `setuptools` | ≥18.5 | Cryptography build requirement |
| PyPI | `ipython` | (any) | Interactive shell for test environment |
| PyPI | `cryptography` | (any) | TLS test support |
| PyPI | `coverage[toml]` | (any) | Code coverage measurement |
| PyPI | `python-can` | (any) | CAN bus socket backend for automotive tests |
| PyPI | `brotli` | (any) | Brotli compression support (Linux/macOS only) |
| PyPI | `zstandard` | (any) | Zstandard compression support (Linux/macOS only) |

**Packages Installed in the Environment**

| Package | Installed Version | Purpose |
|---|---|---|
| `scapy` | 2026.04.16 (editable) | Project under test |
| `ipython` | 9.12.0 | Interactive console |
| `cryptography` | 41.0.7 | TLS layer support (system package) |
| `matplotlib` | 3.10.8 | Visualization |
| `pyx` | 0.17 | PDF/PS diagram rendering |
| `mock` | 5.2.0 | Test mocking |

### 0.3.2 Dependency Updates

This task is a read-only QnA exercise that produces a documentation artifact. No dependency changes are required.

- **No new packages** need to be added to `pyproject.toml`, `setup.py`, or any dependency manifest.
- **No import updates** are needed in any source files.
- **No external reference updates** are required in configuration, documentation, or build files.

The only installation actions performed were:
- `pip install -e ".[all]"` — Install Scapy in editable mode with all optional dependencies for full functionality testing.
- `pip install mock` — Install the `mock` test dependency needed by `test/answering_machines.uts`, which is listed in `tox.ini` deps but not included in the `[all]` optional group.

## 0.4 Integration Analysis

### 0.4.1 Existing Code Touchpoints

Since this is a documentation-only task that produces a QnA markdown file, no source code modifications are required. However, the analysis must deeply inspect and trace through several interconnected code paths. The following touchpoints document the source code areas that must be read, executed, and explained in the output document.

**IP Header Construction Pipeline (Primary Analysis Target)**

The core analysis traces packet construction across three files:

- **`scapy/layers/inet.py` line 521–650**: The `IP` class definition, including `fields_desc` (lines 524–538) which declares all IP header fields with their types and defaults, and `post_build()` (lines 541–553) which auto-computes `ihl`, `len`, and `chksum` when they are `None`.
- **`scapy/packet.py` line 596–607**: The `__div__`/`__truediv__` operators that implement the `IP()/ICMP()` stacking syntax by cloning both layers and calling `add_payload()`.
- **`scapy/packet.py` line 360–378**: The `add_payload()` method which, upon attaching ICMP as IP's payload, checks `ICMP._overload_fields` for `IP` and copies `{'frag': 0, 'proto': 1}` into `IP.overloaded_fields`.
- **`scapy/packet.py` line 438–448**: The `getfieldval()` method which implements the field resolution order: `self.fields` (user-set) → `self.overloaded_fields` (from payload binding) → `self.default_fields` (from `fields_desc`).
- **`scapy/packet.py` line 678–714**: The `self_build()` method which iterates `fields_desc`, calls `getfieldval()` for each field, and serializes them via `addfield()`.
- **`scapy/packet.py` line 724–742**: The `do_build()` method which orchestrates `self_build()` → `post_transforms` → `do_build_payload()` → `post_build()`.
- **`scapy/fields.py` line 854–889**: The `SourceIPField` class whose `__findaddr()` method consults `conf.route.route(dst)` to resolve the source IP address based on the destination.
- **`scapy/layers/inet.py` line 503–518**: The `DestIPField` class which resolves the destination IP, defaulting to `"127.0.0.1"`.

**Layer Binding Integration**

- **`scapy/layers/inet.py` line 1112**: `bind_layers(IP, ICMP, frag=0, proto=1)` — This call at module load time establishes the bidirectional binding between IP and ICMP, which causes IP's `proto` field to be overloaded to `1` (ICMP) and `frag` to `0` whenever an ICMP payload is attached.
- **`scapy/packet.py` line 1953–1971**: `bind_top_down()` stores the field overload values in `ICMP._overload_fields[IP]`, creating the mechanism by which ICMP "tells" its parent IP layer what protocol number to use.

**Network Transmission Integration**

- **`scapy/sendrecv.py` line 422**: `send()` function opens a `conf.L3socket` (defaults to `L3PacketSocket` on Linux), builds the packet via `build()`, and transmits it.
- **`scapy/sendrecv.py` line 656**: `sr1()` combines send and receive, using `SndRcvHandler` (line 100) to match responses via the `hashret()`/`answers()` method pair.
- **`scapy/arch/linux.py`**: `L3PacketSocket` uses PF_PACKET sockets, which on the loopback interface exhibit the behavior where the kernel's ICMP echo-reply may not be matched by Scapy's response-matching logic due to loopback packet duplication.
- **`scapy/supersocket.py`**: `L3RawSocket` as an alternative that uses AF_INET raw sockets, which correctly receives ICMP echo-replies on loopback.

**Interactive Console Integration**

- **`scapy/main.py` line 503**: `interact()` entry point, registered as CLI command `scapy` via `pyproject.toml`.
- **`scapy/main.py` lines 593–656**: Banner construction logic: ASCII logo array, version insertion, terminal width detection, quote selection, IPython detection.
- **`scapy/__init__.py` lines 124–168**: Version resolution chain used by `conf.version`.

**Test Framework Integration**

- **`scapy/tools/UTscapy.py`**: Test campaign parser and executor; reads `.utsc` config files, expands test file globs, applies keyword filters, runs tests in sequence.
- **`test/configs/linux.utsc`**: Defines the test matrix: 12 test file patterns, exclusions (`windows.uts`, `bpf.uts`), keyword exclusions (`osx`, `windows`, `ipv6`), `breakfailed: true` which halts after the first campaign with failures.

## 0.5 Technical Implementation

### 0.5.1 File-by-File Execution Plan

This task produces a single output file by reading, executing, and analyzing existing source code. No source modifications are made.

**Group 1 — Output Artifact**

- **CREATE**: `blitzy/documentation/scapy_0925ada48540.md` — The comprehensive QnA markdown document. This is the sole deliverable. It must include all captured outputs, source code explanations, and test results organized as answers to each of the user's questions.

**Group 2 — Source Files to Read and Analyze (Read-Only)**

- **READ**: `scapy/__init__.py` — Trace the `_version()` function to explain version resolution at this commit.
- **READ**: `scapy/main.py` — Analyze the `interact()` function for startup banner construction logic.
- **READ**: `scapy/packet.py` — Trace `__div__()`, `add_payload()`, `getfieldval()`, `self_build()`, `do_build()`, `post_build()`, `show()`, `show2()` for packet construction explanation.
- **READ**: `scapy/fields.py` — Analyze `SourceIPField`, `IPField`, `BitField`, `ShortField`, and other field types used in IP header.
- **READ**: `scapy/layers/inet.py` — Analyze `IP` class `fields_desc`, `post_build()`, `ICMP` class, and `bind_layers()` call.
- **READ**: `scapy/sendrecv.py` — Understand `send()`, `sr1()`, `sniff()` for packet transmission analysis.
- **READ**: `scapy/arch/linux.py` — Understand `L3PacketSocket` PF_PACKET behavior on loopback.
- **READ**: `scapy/config.py` — Understand `conf` defaults for socket selection, routing.
- **READ**: `scapy/route.py` — Understand routing table lookup for source IP auto-resolution.

**Group 3 — Runtime Execution (Captured Output)**

- **EXECUTE**: `from scapy.all import *; IP()/ICMP()` — Capture packet construction output.
- **EXECUTE**: `(IP()/ICMP()).show()` — Capture `show()` output showing default and user-set fields.
- **EXECUTE**: `(IP()/ICMP()).show2()` — Capture `show2()` output showing fully computed fields.
- **EXECUTE**: `send(IP(dst='127.0.0.1')/ICMP())` — Transmit ICMP packet and capture network behavior.
- **EXECUTE**: `sr1(IP(dst='127.0.0.1')/ICMP())` with both `L3PacketSocket` and `L3RawSocket` — Demonstrate send/receive behavior on loopback.
- **EXECUTE**: `sniff(iface='lo', ...)` — Capture loopback traffic to observe what the kernel does.
- **EXECUTE**: `python3 -m scapy.tools.UTscapy -c ./test/configs/linux.utsc -N -f text` — Run full test suite and capture results.

**Group 4 — Temporary Script Management**

- Any temporary Python scripts created for exploration must be deleted after output capture.
- The final state of the repository must contain no new files except `blitzy/documentation/scapy_0925ada48540.md`.

### 0.5.2 Implementation Approach

The implementation follows a systematic capture-then-document approach:

- **Establish feature foundation** by installing Scapy in editable mode with all optional dependencies, verifying the installation produces a working interactive environment.
- **Capture runtime outputs** by executing each user-requested operation (shell startup, ICMP creation, show() calls, packet transmission, test suite) and recording stdout/stderr verbatim.
- **Trace source code paths** by reading the relevant Python modules to explain *how* Scapy constructs the IP header — from the `fields_desc` declaration through `post_build()` auto-computation — providing file paths and line numbers as evidence.
- **Analyze network behavior** by describing what happens at the network layer when an ICMP packet is sent to localhost: the PF_PACKET socket sends via the loopback interface, the Linux kernel receives the echo-request and generates an echo-reply, and Scapy's response-matching behavior differs between `L3PacketSocket` and `L3RawSocket`.
- **Synthesize findings** into a structured markdown document with clear sections, code blocks for captured output, and explanatory prose with rationale.

### 0.5.3 Key Technical Findings to Document

The following findings from the exploration must be documented in the output markdown:

**Version Resolution**:
- At commit `0925ada4`, the shallow git clone has no tags, so `_version_from_git_describe()` fails.
- The `_version_from_git_archive()` also fails because the `$Format:...$` placeholders are unexpanded.
- The fallback chain reaches the file modification timestamp, producing a date-based version like `2026.04.16`.

**IP Header Auto-Population**:
- Fields set to concrete defaults in `fields_desc`: `version=4`, `tos=0x0`, `id=1`, `flags=0`, `frag=0`, `ttl=64`.
- Fields left as `None` (deferred to `post_build()`): `ihl`, `len`, `chksum`.
- Fields auto-resolved dynamically: `src` (via `SourceIPField.__findaddr()` consulting the routing table), `dst` (defaults to `127.0.0.1`).
- Fields injected via `overload_fields` from ICMP binding: `proto=1` (ICMP), `frag=0`.
- After `show2()` (full build): `ihl=5`, `len=28`, `chksum=0x7cde`.

**Loopback Transmission Behavior**:
- `send()` with default `L3PacketSocket` transmits successfully (1 packet sent).
- `sr1()` with `L3PacketSocket` on loopback: Receives 1 packet but gets 0 answers (PF_PACKET duplication behavior on `lo`).
- `sr1()` with `L3RawSocket`: Successfully receives the kernel's ICMP echo-reply (`type=echo-reply`, `chksum=0x0`, `id=50929`).
- `sniff(iface='lo')` captures 2 identical copies of the echo-request on the loopback (one from send, one looped back).

**Test Suite Results**:
- 12 test campaigns loaded from `test/configs/linux.utsc`.
- **555 tests passed**, **14 tests failed** across all campaigns.
- All 14 failures are in `test/regression.uts`, caused by missing external tools (`tshark`, `libpcap.so`, `tcpdump`) and the `mock` module loading order.
- The `breakfailed: true` config setting stops execution after the first failed campaign, so layer-specific, contrib, and tool tests were not reached.

## 0.6 Scope Boundaries

### 0.6.1 Exhaustively In Scope

**Output Artifact**
- `blitzy/documentation/scapy_0925ada48540.md` — The sole deliverable

**Source Files to Analyze (Read-Only)**
- `scapy/__init__.py` — Version resolution mechanism
- `scapy/main.py` — Interactive console startup, banner logic
- `scapy/packet.py` — Packet base class, build pipeline, show methods, binding system
- `scapy/fields.py` — SourceIPField, IPField, and all field types used in IP header
- `scapy/layers/inet.py` — IP class, ICMP class, bind_layers, post_build
- `scapy/sendrecv.py` — send(), sr1(), sr(), sniff() implementations
- `scapy/supersocket.py` — Socket abstraction layer
- `scapy/arch/linux.py` — L3PacketSocket, PF_PACKET implementation
- `scapy/config.py` — conf singleton, socket backend defaults, load_layers
- `scapy/route.py` — IPv4 routing table for source IP resolution
- `scapy/all.py` — Convenience import module
- `scapy/themes.py` — Color theme for banner rendering

**Test Files to Execute**
- `scapy/tools/UTscapy.py` — Test runner
- `test/configs/linux.utsc` — Linux non-root test configuration
- `test/*.uts` — Core test campaigns
- `test/scapy/layers/*.uts` — Layer test campaigns
- `test/contrib/**/*.uts` — Contrib test campaigns
- `test/tools/*.uts` — Tool test campaigns

**Configuration and Metadata Files to Reference**
- `pyproject.toml` — Python version constraints, optional dependencies, entry points
- `setup.py` — Build configuration and version injection
- `tox.ini` — Test matrix and dependency declarations
- `README.md` — Project overview documentation

**Runtime Operations to Capture**
- Scapy shell startup banner and version
- `IP()/ICMP()` packet creation with `show()` and `show2()` output
- `send()` and `sr1()` to `127.0.0.1` with loopback traffic capture
- Full UTscapy test suite execution with pass/fail reporting

### 0.6.2 Explicitly Out of Scope

- **Source Code Modifications**: No files in `scapy/`, `test/`, `doc/`, or root directory will be modified, per the user's explicit instruction and the `SWE-AtlasQnA-Repo` implementation rule.
- **New Source Code**: No Python modules, test files, or configuration files will be added to the repository besides the output markdown document.
- **Root-Privilege Tests**: Tests requiring root access (e.g., raw socket tests filtered by `-N` flag) are out of scope due to container privilege constraints.
- **External Tool Installation**: Installing `tshark`, `tcpdump`, `libpcap`, or other external tools to fix the 14 test failures is out of scope — the test results are reported as-is.
- **Performance Optimization**: No performance improvements to Scapy's packet construction or test execution.
- **Protocol Extensions**: No new protocol layers or contrib modules.
- **Windows/macOS Analysis**: Platform-specific behavior is out of scope; analysis is limited to the Linux environment.
- **TLS/SSL Deep Analysis**: While `cryptography` is installed, deep TLS subsystem exploration is not part of the user's questions.
- **Automotive Subsystem**: The automotive protocol stack (UDS, DoIP, ISO-TP) is not relevant to the user's questions.
- **Persistent Environment Changes**: Any temporary scripts or files created during exploration must be cleaned up; no persistent changes to the environment beyond the output document.

## 0.7 Rules for Feature Addition

The following rules are explicitly emphasized by the user and the project's implementation directives:

- **No Source File Modification**: The user states: *"Don't modify any source files."* All existing files in the Scapy repository (`scapy/`, `test/`, `doc/`, root configs) must remain untouched. This is reinforced by the `SWE-AtlasQnA-Repo` rule: *"Do not modify any existing files in the source repository."*
- **Temporary Script Cleanup**: The user instructs: *"if you need to create temp scripts, just clean them up after."* Any Python helper scripts created during exploration must be deleted after their output is captured. The final repository state must contain only the original files plus the output markdown document.
- **Single Output Document**: Per `SWE-AtlasQnA-Repo`: *"Create a new markdown document named `<source_branch_name>.md`."* The document must be named `scapy_0925ada48540.md` (matching the branch name `scapy_0925ada48540`).
- **Document Placement**: Per `SWE-AtlasQnA-Repo`: *"Place the generated document in the `blitzy/documentation` directory."* The full path is `blitzy/documentation/scapy_0925ada48540.md`.
- **No Additional Code**: Per `SWE-AtlasQnA-Repo`: *"Do not add any other code in the source repository (besides the above requested document)."*
- **Evidence-Based Answers**: Per `SWE-AtlasQnA-Repo`: *"Do not make assumptions, base your answers on the code as the truth."* Every claim in the document must be backed by source code evidence (file paths and line numbers) or captured runtime output.
- **Rationale Inclusion**: Per `SWE-AtlasQnA-Repo`: *"Provide thinking / rationale behind the answers."* The document must explain the *why* behind observed behaviors, not merely report outputs.
- **Build and Run Verification**: Per `SWE-AtlasQnA-Repo`: *"Build and run the source code to analyse the repository behavior as needed."* The Scapy project must be installed and executed to produce real captured outputs for all runtime questions.

## 0.8 References

### 0.8.1 Repository Files and Folders Searched

The following files and directories were inspected during context gathering to derive the conclusions in this Agent Action Plan:

**Core Source Files Inspected**

| File Path | Lines Examined | Purpose |
|---|---|---|
| `scapy/__init__.py` | 1–170 | Version resolution chain (`_version()`, `_version_from_git_describe()`, `_version_from_git_archive()`) |
| `scapy/main.py` | 503–665 | `interact()` function, banner construction, logo arrays, quote selection, IPython detection |
| `scapy/packet.py` | 77–200, 246–290, 360–380, 438–470, 596–612, 678–790, 1459–1500, 1950–2040 | `Packet.__init__()`, `init_fields()`, `add_payload()`, `getfieldval()`, `__div__()`, build pipeline, `show()`/`show2()`, `bind_layers()` |
| `scapy/fields.py` | 854–900 | `SourceIPField` class with `__findaddr()` routing lookup |
| `scapy/layers/inet.py` | 503–520, 521–700, 952–1000, 1108–1112 | `DestIPField`, `IP` class with `fields_desc` and `post_build()`, `ICMP` class, `bind_layers()` calls |
| `scapy/config.py` | (summary) | `Conf` class, socket backend defaults, `load_layers` list |
| `scapy/sendrecv.py` | (summary) | `send()`, `sr1()`, `sniff()` function signatures and behavior |
| `scapy/all.py` | (summary) | Convenience import module |

**Configuration and Build Files Inspected**

| File Path | Purpose |
|---|---|
| `pyproject.toml` | Python version constraint (`>=3.7, <4`), entry points, optional dependencies, classifiers |
| `setup.py` | Build configuration, version injection via `_build_version()`, Python 2 rejection |
| `tox.ini` | Test matrix (py37–py311), test environments, dependency lists, test commands |
| `.appveyor.yml` | Windows CI configuration |
| `.travis.yml` | Legacy CI configuration |
| `.gitattributes` | Git archive export-subst configuration |

**Test Infrastructure Files Inspected**

| File Path | Purpose |
|---|---|
| `test/configs/linux.utsc` | Linux non-root test configuration: testfile globs, keyword exclusions, `breakfailed` setting |
| `test/*.uts` | 17 core test campaign files listed |
| `test/scapy/layers/*.uts` | 40+ layer-specific test files enumerated |
| `test/contrib/**/*.uts` | 100+ contrib test files enumerated |
| `test/tools/*.uts` | 3 tool test files enumerated |

**Directories Traversed**

| Directory | Depth | Contents |
|---|---|---|
| `/tmp/blitzy/scapy/scapy_0925ada48540_ca08b7/` | Root | Project root with 8 directories and 11 files |
| `scapy/` | 1 | 30+ core modules and 9 subdirectories |
| `scapy/layers/` | 2 | 48 protocol layer modules plus `tls/` subfolder |
| `scapy/arch/` | 2 | Platform abstraction (linux.py, bpf/, windows/) |
| `scapy/contrib/` | 2 | 90+ contrib modules across multiple subdirectories |
| `scapy/tools/` | 2 | UTscapy.py and automotive scanner tools |
| `test/` | 1 | Test campaigns and config directory |
| `test/configs/` | 2 | 7 platform-specific test configuration files |
| `.git/` | 1 | Git metadata (commit `0925ada4`, branch `scapy_0925ada48540`) |

### 0.8.2 Runtime Executions Performed

| Execution | Purpose | Key Outcome |
|---|---|---|
| `pip install -e ".[all]"` | Install Scapy with all optional dependencies | Scapy version resolves to `2026.04.16` |
| `pip install mock` | Install test dependency | Required for `test/answering_machines.uts` |
| `python3 -c "from scapy.all import *; (IP()/ICMP()).show()"` | Capture show() output | Displayed default IP/ICMP fields with `None` deferred values |
| `python3 -c "from scapy.all import *; (IP()/ICMP()).show2()"` | Capture show2() output | Displayed fully computed fields: `ihl=5`, `len=28`, `chksum=0x7cde` |
| `python3 -c "... send(IP(dst='127.0.0.1')/ICMP()) ..."` | Transmit ICMP to localhost | 1 packet sent successfully |
| `python3 -c "... sr1(..., L3RawSocket) ..."` | Send/receive with raw socket | Received ICMP echo-reply from kernel |
| `python3 -c "... sniff(iface='lo') ..."` | Capture loopback traffic | 2 packets captured (echo-request duplicated on loopback) |
| `python3 -m scapy.tools.UTscapy -c ./test/configs/linux.utsc -N -f text` | Run full test suite | 555 passed, 14 failed across 12 campaigns |

### 0.8.3 Technical Specification Sections Referenced

| Section | Purpose |
|---|---|
| 1.1 Executive Summary | Project overview, version, license, stakeholders |
| 2.1 Feature Catalog | Feature inventory with module mappings (F-001 through F-020) |
| 3.2 Programming Languages | Python version constraints, standard library usage |
| 4.10 Interactive Console Startup | Console initialization sequence, session persistence |

### 0.8.4 Attachments

No external attachments (Figma URLs, design files, or supplementary documents) were provided for this task.

