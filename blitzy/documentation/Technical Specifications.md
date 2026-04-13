# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

### 0.1.1 Core Feature Objective

Based on the prompt, the Blitzy platform understands that the new feature requirement is to **create a comprehensive runtime investigation document** for the Scapy interactive packet manipulation library. The user is joining a team that relies heavily on Scapy and needs to orient themselves by answering specific empirical questions about how Scapy behaves when running. This is a **read-only, exploratory, documentation-only task** — no source code modifications are permitted.

The specific questions to be answered through runtime investigation are:

- **Startup Banner and Version**: What welcome message and version information does Scapy display when its interactive console starts? This requires examining the `interact()` function in `scapy/main.py` and the version computation pipeline in `scapy/__init__.py`.
- **Loaded Protocol Layer Count**: How many protocol layers are actually loaded and available in the runtime environment — not merely counted as files in the source tree, but registered in `conf.layers` and ready for use? This requires invoking `from scapy.all import conf` and inspecting the `conf.layers` registry.
- **Default Verbosity Level**: What is the startup value of `conf.verb`, what numerical value does it hold, and what does that number mean for output behavior during send/receive operations? This requires examining `scapy/config.py` and `scapy/sendrecv.py`.
- **Networking Socket Implementation**: What socket backend does Scapy select on this Linux system, and what class is used for `conf.L3socket`? This requires inspecting the platform-detection and socket-selection logic in `scapy/config.py` and `scapy/arch/linux.py`.
- **ICMP Ping Packet Structure**: When constructing `IP(dst='192.168.1.1')/ICMP()`, what is the actual composite object type, what layers are involved, and how are they related? This requires running the packet construction and inspecting the resulting object hierarchy.
- **Default Theme**: What terminal output theme is active in a default interactive session, and what is it called in the code? This requires examining `scapy/themes.py` and the theme assignment in `scapy/main.py`'s `interact()` function.

Implicit requirements detected:

- The answers must be based on **actual runtime execution**, not merely reading source files, since the user explicitly distinguished between "files in the source tree" and "what's actually loaded and ready to use."
- A markdown document named `scapy_0925ada48540.md` must be placed in the `blitzy/documentation` directory, as mandated by the project implementation rule `SWE-AtlasQnA-Repo`.
- The document must include rationale and thinking behind the answers, not just bare facts.
- No existing repository files may be modified; only the new documentation file is to be created.

### 0.1.2 Special Instructions and Constraints

- **Read-Only Repository Constraint**: The user explicitly stated: "Don't modify any of the repository files while investigating this." The implementation rule `SWE-AtlasQnA-Repo` reinforces this: "Do not modify any existing files in the source repository."
- **Temporary Script Cleanup**: If temporary test scripts are needed for investigation, they must be cleaned up afterward.
- **Evidence-Based Answers**: The implementation rule specifies: "Do not make assumptions, base your answers on the code as the truth."
- **Output Location**: The generated document must be placed at `blitzy/documentation/scapy_0925ada48540.md` (branch name is `scapy_0925ada48540`).
- **Thinking/Rationale**: The implementation rule requires: "Provide thinking / rationale behind the answers."

### 0.1.3 Technical Interpretation

These feature requirements translate to the following technical implementation strategy:

- To **answer the version and banner question**, we will execute Scapy's version resolution pipeline from `scapy/__init__.py` and reconstruct the ASCII-art banner from `scapy/main.py`'s `interact()` function (lines 593–651), which composes the Scapy logo with version/welcome text using `conf.color_theme.logo()` and `conf.color_theme.success()` formatters.
- To **count loaded protocol layers**, we will import `scapy.all` (which triggers the full layer loading pipeline through `scapy/layers/all.py` iterating over `conf.load_layers`) and query `len(conf.layers)` for the actual registered count.
- To **determine verbosity behavior**, we will inspect `conf.verb` (defined at line 759 of `scapy/config.py` with default value `2`) and trace how verbosity levels 0–3 control output in `scapy/sendrecv.py`.
- To **identify the socket implementation**, we will check `conf.L3socket` after the platform-detection pipeline in `scapy/config.py`'s `_set_conf_sockets()` function selects the Linux backend.
- To **analyze ICMP packet structure**, we will construct `IP(dst='192.168.1.1')/ICMP()` and introspect the resulting linked-list object hierarchy using `.payload`, `.underlayer`, `type()`, and `.show()`.
- To **identify the default theme**, we will check both the non-interactive default (`NoTheme` from `scapy/config.py` line 818) and the interactive-session theme set at `scapy/main.py` line 515 (`DefaultTheme()`).
- To **deliver the output**, we will create the directory `blitzy/documentation/` in the repository and write the investigation results as `scapy_0925ada48540.md`.

## 0.2 Repository Scope Discovery

### 0.2.1 Comprehensive File Analysis

Since this task is a **documentation-only, read-only investigation**, no existing source files are modified. The file analysis below identifies every source file and module that was examined to derive the runtime answers, grouped by investigative purpose.

#### Files Examined for Version and Banner Investigation

| File Path | Purpose of Examination |
|---|---|
| `scapy/__init__.py` | Version computation pipeline: `_version()` tries four methods (env var, VERSION file, git archive, git describe) before falling back to `__init__.py` modification timestamp |
| `scapy/main.py` (lines 49–60) | The `QUOTES` list containing 8 humorous quotes randomly selected for the full-size banner |
| `scapy/main.py` (lines 503–708) | The `interact()` function implementing the banner rendering logic, including the ASCII-art logo (lines 593–618), the text banner (lines 628–639), and the mini-banner variant for narrow terminals |

#### Files Examined for Protocol Layer Count Investigation

| File Path | Purpose of Examination |
|---|---|
| `scapy/config.py` (lines 720–760) | `Conf` class definition with `layers = LayersList()` registry |
| `scapy/config.py` (lines 848–897) | `load_layers` list defining the 48 default layer module names |
| `scapy/layers/all.py` | Layer-loading orchestrator that iterates `conf.load_layers` and calls `load_layer()` for each |
| `scapy/arch/__init__.py` (line 149) | Platform-conditional addition of `tuntap` to `conf.load_layers` on Linux/BSD |
| `scapy/layers/bluetooth4LE.py` | Imports from `scapy.contrib.ethercat`, causing transitive loading of contrib layers |
| `scapy/layers/dcerpc.py` | Imports from `scapy.contrib.rtps.common_types`, causing transitive loading of RTPS contrib layers |
| `scapy/all.py` | The umbrella import module that triggers the entire layer-loading pipeline |

#### Files Examined for Verbosity and Socket Investigation

| File Path | Purpose of Examination |
|---|---|
| `scapy/config.py` (line 759) | Default `verb = 2` definition with comment: "level of verbosity, from 0 (almost mute) to 3 (verbose)" |
| `scapy/sendrecv.py` (lines 132–133, 216, 238, 249, 282, 297, 354–355, 379, 391, 552, 729–756) | Verbosity-gated output behavior at levels 0, 1, 2, and 3 |
| `scapy/config.py` (lines 608–675) | `_set_conf_sockets()` function implementing platform-based socket backend selection |
| `scapy/arch/linux.py` (lines 475–476, 587–588) | `L2Socket` and `L3PacketSocket` class definitions with `desc` strings |
| `scapy/supersocket.py` (lines 50–56) | `_SuperSocket_metaclass.__repr__()` showing how socket classes format their description |

#### Files Examined for ICMP Packet Structure Investigation

| File Path | Purpose of Examination |
|---|---|
| `scapy/packet.py` | `Packet` base class with `__truediv__()` operator (layer stacking), `payload`/`underlayer` linked-list chain, `add_payload()`, `show()`, `summary()` |
| `scapy/layers/inet.py` | `IP` and `ICMP` class definitions with `bind_layers(IP, ICMP, proto=1)` |
| `scapy/base_classes.py` | Metaclass machinery (`Packet_metaclass`) for auto-registration in `conf.layers` |

#### Files Examined for Theme Investigation

| File Path | Purpose of Examination |
|---|---|
| `scapy/themes.py` | All 13 theme classes: `ColorTheme`, `NoTheme`, `AnsiColorTheme`, `BlackAndWhite`, `DefaultTheme`, `BrightTheme`, `RastaTheme`, `ColorOnBlackTheme`, `FormatTheme`, `LatexTheme`, `LatexTheme2`, `HTMLTheme`, `HTMLTheme2` |
| `scapy/config.py` (line 818) | Initial `color_theme = Interceptor("color_theme", NoTheme(), _prompt_changer)` — non-interactive default |
| `scapy/main.py` (line 515) | `conf.color_theme = DefaultTheme()` — set during interactive session startup |

### 0.2.2 Web Search Research Conducted

No web searches were required for this task. All answers were derived directly from the source code and verified through runtime execution of the Scapy library from the cloned repository at commit `0925ada4`.

### 0.2.3 New File Requirements

A single new file must be created:

| File Path | Purpose |
|---|---|
| `blitzy/documentation/scapy_0925ada48540.md` | Comprehensive markdown document answering the user's runtime investigation questions, including rationale and evidence from both source code analysis and runtime execution |

This file does not modify any existing repository files and is placed in a dedicated documentation directory as required by the `SWE-AtlasQnA-Repo` implementation rule.

## 0.3 Dependency Inventory

### 0.3.1 Private and Public Packages

Since this is a documentation-only task with no code modifications to the Scapy codebase, no new dependencies are introduced. The following table documents the key packages relevant to executing the runtime investigation:

| Registry | Package | Version | Purpose |
|---|---|---|---|
| PyPI | `scapy` | 2026.04.13 (dev, from git) | The Scapy library itself, installed in editable mode from the local repository clone at commit `0925ada4` |
| Python stdlib | `socket` | (bundled with Python 3.12.3) | Provides `socket.has_ipv6` for IPv6 detection and raw socket primitives used by `scapy/arch/linux.py` |
| Python stdlib | `ctypes` | (bundled with Python 3.12.3) | Used by `scapy/supersocket.py` for `tpacket_auxdata` struct definitions and BPF-related structures |
| System | `python3` | 3.12.3 | Runtime interpreter; Scapy requires `>=3.7, <4` per `pyproject.toml` |

The Scapy version `2026.04.13` is a dynamically computed development version. The version pipeline in `scapy/__init__.py` tried four methods in order:

- `SCAPY_VERSION` environment variable — not set
- `scapy/VERSION` file — does not exist in this checkout
- Git archive substitution — not applicable (not a git archive)
- `git describe --tags --always --long` — returned `0925ada4` (no tags in repo), then attempted `git rev-list --tags --max-count=1` which also produced no usable result
- **Fallback**: modification timestamp of `scapy/__init__.py` formatted as `YYYY.MM.DD` → `2026.04.13`

### 0.3.2 Dependency Updates

No dependency updates are required. This task creates a single markdown file and does not alter `pyproject.toml`, `setup.py`, `tox.ini`, or any other dependency manifest.

No import updates, external reference updates, or build file modifications are needed.

## 0.4 Integration Analysis

### 0.4.1 Existing Code Touchpoints

This task has **zero existing code touchpoints**. No source files are modified, no new protocol layers are registered, and no configuration changes are made. The investigation is entirely observational.

However, understanding the integration points that were **examined** during the investigation is critical for documenting the rationale behind each answer:

#### Version Resolution Chain

The version displayed in the startup banner is resolved through a chain of modules:

- `scapy/__init__.py` → `_version()` → tries `os.environ['SCAPY_VERSION']`, then `scapy/VERSION` file, then `_version_from_git_archive()`, then `_version_from_git_describe()`, then modification-timestamp fallback
- `scapy/config.py` (line 726) → `version = ReadOnlyAttribute("version", VERSION)` — exposes the computed version on the `conf` singleton
- `scapy/main.py` (line 633) → `"   | Version %s" % conf.version` — embeds the version into the ASCII banner

#### Layer Registration Pipeline

Protocol layers are registered in `conf.layers` through a multi-stage pipeline:

- `scapy/config.py` (lines 848–897) → `load_layers` list defines 48 default modules
- `scapy/arch/__init__.py` (line 149) → Adds `tuntap` on Linux/BSD, bringing the total to 49 modules
- `scapy/layers/all.py` → Iterates `conf.load_layers` and calls `load_layer()` for each
- `scapy/base_classes.py` → `Packet_metaclass.__new__()` auto-registers each `Packet` subclass into `conf.layers` at import time
- Transitive imports (e.g., `bluetooth4LE` → `scapy.contrib.ethercat`, `dcerpc` → `scapy.contrib.rtps`) cause additional contrib modules to register their classes

#### Socket Selection Pipeline

The L3 socket backend is selected through:

- `scapy/consts.py` → Detects `LINUX = True` via `sys.platform`
- `scapy/config.py` → `_set_conf_sockets()` (lines 608–675) evaluates platform flags
- `scapy/arch/__init__.py` (line 151) → `_set_conf_sockets()` is called, which imports `L3PacketSocket` from `scapy/arch/linux.py` and assigns it to `conf.L3socket`

### 0.4.2 Dependency Injections

No service registrations, dependency injections, or container modifications are required for this documentation-only task.

### 0.4.3 Database/Schema Updates

No database or schema changes are required. This task produces only a single markdown document.

## 0.5 Technical Implementation

### 0.5.1 File-by-File Execution Plan

This task requires creating exactly one new file and modifying zero existing files:

- **CREATE**: `blitzy/documentation/scapy_0925ada48540.md` — A comprehensive markdown document answering six specific runtime investigation questions about the Scapy environment, with rationale grounded in source code evidence and runtime execution results.

The directory `blitzy/documentation/` must be created if it does not already exist.

### 0.5.2 Implementation Approach

The implementation proceeds through a single-phase approach:

**Phase 1 — Runtime Investigation and Document Generation**

Establish the answers by executing Scapy in a non-interactive Python session and recording the results:

- **Version and Banner**: Execute `from scapy.config import conf; print(conf.version)` and reconstruct the banner by importing the logo and banner arrays from `scapy/main.py` (lines 593–639). The banner is composed by zipping the ASCII logo lines with the text banner lines and applying `conf.color_theme.logo()` and `conf.color_theme.success()` formatters.
- **Protocol Layer Count**: Execute `from scapy.all import conf; print(len(conf.layers))` to obtain the actual registered count (runtime result: **1319 protocol layer classes** across 54 unique modules from 49 loaded layer modules).
- **Verbosity Level**: Read `conf.verb` (runtime result: **2**) and document the behavior at each level by tracing usage in `scapy/sendrecv.py`.
- **Socket Implementation**: Read `conf.L3socket` (runtime result: **`L3PacketSocket`** from `scapy.arch.linux`, using Linux PF_PACKET sockets).
- **ICMP Packet Structure**: Construct `IP(dst='192.168.1.1')/ICMP()` and introspect: the result is a **linked-list** of `Packet` objects chained via `.payload`/`.underlayer` references, with the outermost type being `IP`.
- **Default Theme**: Check `conf.color_theme` in both non-interactive (result: `NoTheme`) and interactive mode (result: `DefaultTheme`, set at line 515 of `scapy/main.py`).

**Phase 2 — Document Assembly**

Write the markdown document to `blitzy/documentation/scapy_0925ada48540.md` containing:

- A question-and-answer structure addressing each of the user's six questions
- Source code references with file paths and line numbers
- Runtime execution evidence showing actual outputs
- Rationale explaining why each answer is what it is, grounded in the code

### 0.5.3 Key Runtime Investigation Results Summary

The following table summarizes the concrete findings that will be documented:

| Question | Answer | Source Evidence |
|---|---|---|
| Welcome message | ASCII-art Scapy logo + "Welcome to Scapy / Version {version}" + random quote + "Have fun!" | `scapy/main.py` lines 593–651 |
| Version | `2026.04.13` (fallback from `__init__.py` modification timestamp) | `scapy/__init__.py` `_version()` function |
| Loaded protocol layers | **1319** registered `Packet` subclasses across 54 modules | `conf.layers` runtime query |
| Default verbosity | `conf.verb = 2` — "level of verbosity, from 0 (almost mute) to 3 (verbose)" | `scapy/config.py` line 759 |
| Verbosity 2 behavior | At level 2, send/receive functions print dots (`.`) for unmatched and asterisks (`*`) for matched packets, plus summary counts | `scapy/sendrecv.py` lines 282, 297 |
| Socket implementation | `L3PacketSocket` from `scapy.arch.linux` — "read/write packets at layer 3 using Linux PF_PACKET sockets" | `conf.L3socket` runtime query; `scapy/arch/linux.py` line 588 |
| ICMP ping packet structure | Linked-list of `IP` → `ICMP` → `NoPayload`, connected via `.payload`/`.underlayer`; outer type is `IP` | Runtime construction of `IP()/ICMP()` |
| Default theme (interactive) | `DefaultTheme` (class in `scapy/themes.py`) | `scapy/main.py` line 515 |
| Default theme (non-interactive) | `NoTheme` (set in `scapy/config.py` line 818) | `conf.color_theme` before `interact()` |

## 0.6 Scope Boundaries

### 0.6.1 Exhaustively In Scope

- **New documentation file**:
  - `blitzy/documentation/scapy_0925ada48540.md` — the sole deliverable of this task

- **Files read (not modified) for investigation**:
  - `scapy/__init__.py` — version computation
  - `scapy/main.py` — interactive startup, banner, quotes
  - `scapy/config.py` — `Conf` class, `verb`, `color_theme`, `load_layers`, `L3socket`, `_set_conf_sockets()`
  - `scapy/themes.py` — all theme class definitions (`NoTheme`, `DefaultTheme`, `AnsiColorTheme`, etc.)
  - `scapy/sendrecv.py` — verbosity-gated output behavior
  - `scapy/packet.py` — `Packet` base class, `__truediv__`, payload chaining
  - `scapy/layers/inet.py` — `IP` and `ICMP` class definitions
  - `scapy/arch/linux.py` — `L3PacketSocket`, `L2Socket` definitions
  - `scapy/arch/__init__.py` — platform-conditional `tuntap` loading, `_set_conf_sockets()` call
  - `scapy/supersocket.py` — `SuperSocket` base class, `_SuperSocket_metaclass`
  - `scapy/base_classes.py` — `Packet_metaclass` for layer auto-registration
  - `scapy/all.py` — umbrella import triggering full layer loading
  - `scapy/layers/all.py` — layer-loading orchestrator
  - `scapy/consts.py` — platform detection constants
  - `pyproject.toml` — project metadata, Python version constraints
  - `tox.ini` — test matrix, CI configuration
  - `setup.py` — legacy build script

- **Runtime investigation commands executed**:
  - `from scapy.all import conf` — trigger full layer loading
  - `len(conf.layers)` — count loaded protocol layers
  - `conf.verb` — check verbosity
  - `conf.L3socket` — check socket backend
  - `IP(dst='192.168.1.1')/ICMP()` — construct and introspect ICMP ping packet
  - `conf.color_theme` — check theme before and after interactive mode

### 0.6.2 Explicitly Out of Scope

- **Modifying any existing repository file** — explicitly prohibited by the user and the `SWE-AtlasQnA-Repo` rule
- **Adding new Python code to the Scapy package** — the `SWE-AtlasQnA-Repo` rule states "Do not add any other code in the source repository (besides the above requested document)"
- **Sending actual network packets** — the investigation only constructs and introspects packet objects, no traffic is transmitted
- **Installing optional dependencies** (cryptography, IPython, matplotlib) — only the core Scapy library is needed for the investigation
- **Performance benchmarking or profiling** — not requested
- **Modifying Scapy configuration** — investigation is observational only
- **Investigating contrib layers beyond what is transitively loaded** — the user asked about default-loaded layers
- **Windows, macOS, or BSD platform analysis** — the investigation runs on Linux and documents Linux-specific results

## 0.7 Rules for Feature Addition

The following rules are explicitly emphasized by the user and the project implementation rules:

- **SWE-AtlasQnA-Repo Rule**: Create a new markdown document named `<source_branch_name>.md` (i.e., `scapy_0925ada48540.md`) that comprehensively answers the question(s) posed in the prompt. The document must:
  - Provide thinking and rationale behind the answers
  - Be based on the code as the truth, with no assumptions
  - Not modify any existing files in the source repository
  - Not add any other code in the source repository besides the requested document
  - Be placed in the `blitzy/documentation` directory

- **User-Specified Read-Only Constraint**: "Don't modify any of the repository files while investigating this. If you need to write temporary test scripts or commands to figure things out, that's fine, but keep the actual codebase unchanged, and clean up any temporary files or scripts you create when you're done."

- **Evidence-Based Answers**: All answers must be derived from actual runtime execution and source code examination, not from external documentation or assumptions. The user explicitly distinguished between "files in the source tree" and "what's actually loaded and ready to use."

- **Cleanup Obligation**: Any temporary files created during investigation must be removed. The final state of the repository should contain only the original files plus the new `blitzy/documentation/scapy_0925ada48540.md` file.

## 0.8 References

### 0.8.1 Files and Folders Searched

The following files and folders were examined during the investigation to derive the conclusions documented in this Agent Action Plan:

**Core Runtime Files:**
- `scapy/__init__.py` — Version computation pipeline (`_version()`, `_version_from_git_describe()`, `_version_from_git_archive()`, `_parse_tag()`)
- `scapy/__main__.py` — Module execution entry point
- `scapy/main.py` — Interactive console startup (`interact()`), banner rendering, `QUOTES` list, session persistence
- `scapy/config.py` — `Conf` class definition, `verb`, `color_theme`, `load_layers`, socket configuration, `_set_conf_sockets()`
- `scapy/consts.py` — Platform detection constants (`LINUX`, `BSD`, `WINDOWS`, etc.)
- `scapy/themes.py` — All 13 theme classes: `ColorTable`, `ColorTheme`, `NoTheme`, `AnsiColorTheme`, `BlackAndWhite`, `DefaultTheme`, `BrightTheme`, `RastaTheme`, `ColorOnBlackTheme`, `FormatTheme`, `LatexTheme`, `LatexTheme2`, `HTMLTheme`, `HTMLTheme2`
- `scapy/all.py` — Umbrella import surface triggering full layer loading
- `scapy/packet.py` — `Packet` base class, `__truediv__()` operator for layer stacking, payload chaining
- `scapy/fields.py` — Field type system
- `scapy/base_classes.py` — `Packet_metaclass` for auto-registration in `conf.layers`
- `scapy/sendrecv.py` — Send/receive functions with verbosity-gated output logic
- `scapy/supersocket.py` — `SuperSocket` base class, `_SuperSocket_metaclass.__repr__()`

**Protocol Layer Files:**
- `scapy/layers/all.py` — Layer-loading orchestrator
- `scapy/layers/inet.py` — `IP`, `ICMP`, `TCP`, `UDP` protocol definitions
- `scapy/layers/bluetooth4LE.py` — Examined for transitive dependency on `scapy.contrib.ethercat`
- `scapy/layers/dcerpc.py` — Examined for transitive dependency on `scapy.contrib.rtps`

**Platform Abstraction Files:**
- `scapy/arch/__init__.py` — Platform-conditional imports and `tuntap` layer registration
- `scapy/arch/linux.py` — `L3PacketSocket`, `L2Socket`, `L2ListenSocket` definitions
- `scapy/arch/bpf/` — BSD/macOS BPF socket backend (examined for comparison)
- `scapy/arch/windows/` — Windows socket backend (examined for comparison)

**Build and Configuration Files:**
- `pyproject.toml` — Project metadata, Python version constraints (`>=3.7, <4`), entry points, optional dependencies
- `setup.py` — Legacy setuptools build script with Python 2 guard
- `tox.ini` — Testing matrix (py27–py311), QA environments, flake8 config

**Folders Explored:**
- Root (`/`) — Repository structure overview
- `scapy/` — Core package contents (34 files, 7 subfolders)
- `scapy/layers/` — 58 files including TLS subfolder
- `scapy/arch/` — Platform backends (5 files, 2 subfolders)
- `scapy/contrib/` — Optional extensions (transitively loaded modules examined)
- `test/` — Test support area
- `.config/` — CI and automation support

### 0.8.2 Attachments

No attachments were provided for this project. No Figma URLs or external design assets are referenced.

### 0.8.3 External Resources

No external web resources were required. All findings were derived from the local repository clone at commit `0925ada4` ("Add AUTOSAR PDUTransport/PDU with initial batch of tests (#3933)") on branch `scapy_0925ada48540`.

