# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

### 0.1.1 Core Feature Objective

Based on the prompt, the Blitzy platform understands that the new feature requirement is to **investigate and document Scapy's runtime behavior** during the full lifecycle of packet manipulation — specifically across three key phases: construction, transmission, and capture. The deliverable is a comprehensive markdown knowledge document placed in the `blitzy/documentation` directory, named `scapy_0925ada48540.md`, that serves as an authoritative reference on how Scapy operates internally during these operations. No existing repository files shall be modified, and no permanent code artifacts are to be left behind.

The feature requirements, restated with enhanced clarity, are:

- **Packet Construction Observation**: Start the Scapy interactive environment and construct a multi-layer packet (Ethernet / IP / TCP) while observing what Scapy reveals at each step — how each layer is represented, how the `/` stacking operator works, and what the fully assembled packet structure looks like via `show()`, `repr()`, and `summary()`.
- **Packet Transmission Observation**: Send the assembled packet using Scapy's standard sending mechanism (`send()` for L3, `sendp()` for L2) and document what Scapy reports during transmission — routing resolution output, dot-per-packet progress indicators, the "Sent N packets." confirmation message, and any warnings or errors.
- **Packet Sniffing and Dissection Observation**: Capture live traffic using `sniff()` and observe how Scapy presents received raw bytes — how it reconstructs layer hierarchy through the dissection pipeline, which `Packet` class is selected for each layer, and how the final reassembled packet is displayed via `show()` and `summary()`.
- **Summary Documentation**: Produce a clean, self-contained markdown document that synthesizes all observations from building, sending, and sniffing into a coherent narrative grounded in Scapy source code evidence.

Implicit requirements detected:

- The document must reference actual Scapy source code locations (file paths, line numbers, method names) as evidence for behavioral claims.
- Temporary scripts may be used during investigation but must be removed afterward; the repository must remain completely clean.
- Root/admin privileges may be needed for raw socket operations (`send()`, `sniff()`), and the document should note this requirement.
- The implementation rule **SWE-AtlasQnA-Repo** governs the deliverable format: a markdown file named after the source branch (`scapy_0925ada48540.md`) placed in `blitzy/documentation/`.

### 0.1.2 Special Instructions and Constraints

- **Repository Cleanliness**: Per the SWE-AtlasQnA-Repo rule: "Do not modify any existing files in the source repository. Do not add any other code in the source repository (besides the above requested document)." All temporary test scripts must be cleaned up.
- **Evidence-Based Answers**: "Do not make assumptions, base your answers on the code as the truth." Every behavioral claim in the document must be grounded in actual source code evidence from the Scapy repository.
- **Thinking / Rationale**: "Provide thinking / rationale behind the answers." The document must explain *why* Scapy behaves the way it does, not just *what* it does.
- **Destination Directory**: The document goes in `blitzy/documentation/` within the destination repository.
- **Branch-Named File**: The file must be named `scapy_0925ada48540.md` (matching the source branch name `scapy_0925ada48540`).

### 0.1.3 Technical Interpretation

These feature requirements translate to the following technical implementation strategy:

- To **document packet construction behavior**, we will analyze the `Packet.__init__()` method (`scapy/packet.py`, line 141), the `__div__`/`__truediv__` layer-stacking operator (line 596), the `add_payload()` method (line 360), and the display methods `__repr__()` (line 552), `show()` (line 1459), `show2()` (line 1473), and `summary()` (line 1642). We will also analyze the `Ether` class (`scapy/layers/l2.py`, line 244), the `IP` class (`scapy/layers/inet.py`, line 521), and the `TCP` class (line 753), including their `fields_desc`, `post_build()`, and `mysummary()` methods.
- To **document packet transmission behavior**, we will analyze `send()` (`scapy/sendrecv.py`, line 422), `sendp()` (line 452), the internal `__gen_send()` helper (line 332), the `_interface_selection()` function (line 616), the `IP.route()` method (`scapy/layers/inet.py`, line 559), the `Route.route()` method (`scapy/route.py`, line 146), the `L3PacketSocket.send()` method (`scapy/arch/linux.py`, line 598), and the `SuperSocket.send()` method (`scapy/supersocket.py`, line 97).
- To **document packet sniffing behavior**, we will analyze `sniff()` (`scapy/sendrecv.py`, line 1308), the `AsyncSniffer._run()` method (line 1064), the `DefaultSession.on_packet_received()` method (`scapy/sessions.py`, line 96), the `SuperSocket.recv()` method (`scapy/supersocket.py`, line 172), the `Packet.dissect()` method (`scapy/packet.py`, line 1049), `do_dissect()` (line 1002), `do_dissect_payload()` (line 1023), and `guess_payload_class()` (line 1062).
- To **produce the deliverable**, we will create the directory `blitzy/documentation/` and generate `scapy_0925ada48540.md` with structured sections covering each phase, including code references and rationale.


## 0.2 Repository Scope Discovery

### 0.2.1 Comprehensive File Analysis

The Scapy repository is a mature, multi-platform packet manipulation library. The following files and modules are directly relevant to analyzing the runtime behavior of packet construction, transmission, and sniffing. No existing files will be modified — this analysis serves to ground the documentation deliverable in source-code truth.

**Core Packet Engine (Construction Phase)**

| File Path | Relevance | Key Code Points |
|---|---|---|
| `scapy/packet.py` (2553 lines) | Central packet abstraction: construction, stacking, build, dissect, display | `Packet.__init__` (L141), `__div__` (L596), `add_payload` (L360), `build` (L746), `do_build` (L724), `post_build` (L758), `dissect` (L1049), `do_dissect` (L1002), `do_dissect_payload` (L1023), `guess_payload_class` (L1062), `__repr__` (L552), `show` (L1459), `show2` (L1473), `_show_or_dump` (L1383), `summary` (L1642), `_do_summary` (L1617), `mysummary` (L1609), `command` (L1662) |
| `scapy/fields.py` (3868 lines) | Field type system used by all layer definitions | `Field.getfield`, `Field.addfield`, `Field.i2repr`, `Field.any2i` |
| `scapy/base_classes.py` (510 lines) | Metaclass machinery, `Packet_metaclass`, `SetGen`, `Gen` base classes | `Packet_metaclass` — creates `fields_desc` class infrastructure |
| `scapy/volatile.py` (1420 lines) | Lazy/random value generators for deferred field computation | `VolatileValue`, `RandField` — used during `build()` for auto-fields |

**Protocol Layer Definitions (Ether/IP/TCP Stack)**

| File Path | Relevance | Key Code Points |
|---|---|---|
| `scapy/layers/l2.py` (1136 lines) | Ethernet layer, ARP, MAC resolution, neighbor system | `Ether` class (L244), `fields_desc` (L246–248), `DestMACField` (L161), `SourceMACField`, `mysummary` (L262), `Neighbor` class (L94), `getmacbyip` (L122), `inet_register_l3` (L1125), `bind_layers(Ether, IP, type=2048)` (L1101 in inet.py) |
| `scapy/layers/inet.py` (2192 lines) | IP, TCP, UDP, ICMP, bind_layers, checksum, routing | `IP` class (L521), `IP.route` (L559), `IP.post_build` (L539), `IP.mysummary` (L612), `TCP` class (L753), `TCP.post_build` (L767), `TCP.mysummary` (L825), `bind_layers(Ether, IP, type=2048)` (L1101), `bind_layers(IP, TCP, frag=0, proto=6)` (L1113) |
| `scapy/layers/all.py` | Dynamic layer loader — imports all default layers at startup | Reads `conf.load_layers`, calls `load_layer()` for each |

**Send/Receive/Sniff Machinery (Transmission & Capture Phases)**

| File Path | Relevance | Key Code Points |
|---|---|---|
| `scapy/sendrecv.py` (1440 lines) | All send/receive/sniff functions | `send` (L422), `sendp` (L452), `_send` (L396), `__gen_send` (L332), `_interface_selection` (L616), `sr` (L635), `sniff` (L1308), `AsyncSniffer` (L981), `AsyncSniffer._run` (L1064), `SndRcvHandler` (L100), `SndRcvHandler._sndrcv_snd` (L232), `_process_packet` (L270) |
| `scapy/supersocket.py` (548 lines) | Socket abstraction, send/recv at raw level | `SuperSocket.send` (L97), `SuperSocket.recv` (L172), `recv_raw` (L167), `_recv_raw` (L117) |
| `scapy/sessions.py` (397 lines) | Session decoders for sniffing | `DefaultSession.on_packet_received` (L96), `IPSession` (L114), `TCPSession` (L223) |

**Routing and Interface Resolution (Transmission Phase)**

| File Path | Relevance | Key Code Points |
|---|---|---|
| `scapy/route.py` (219 lines) | IPv4 routing table, route resolution | `Route.route` (L146), `Route.__repr__` (L47), `Route.resync` (L41) |
| `scapy/interfaces.py` | Interface registry, `NetworkInterface`, `resolve_iface` | Interface selection and provider system |
| `scapy/arch/linux.py` | Linux-specific PF_PACKET socket implementations | `L3PacketSocket` (L587), `L3PacketSocket.send` (L598), `L2Socket` (L475), `L2ListenSocket` (L579), `recv_raw` (L555) |
| `scapy/arch/__init__.py` | Platform facade, re-exports, `_set_conf_sockets` | Conditionally imports the active backend |

**Interactive Console Startup**

| File Path | Relevance | Key Code Points |
|---|---|---|
| `scapy/main.py` (715 lines) | Interactive console startup | `interact` (L503), `init_session` (L415), `_scapy_builtins` (L302), banner display (L589–655) |
| `scapy/all.py` | Umbrella import — aggregates entire public API | Imports from 25+ modules including `scapy.layers.all` |
| `scapy/themes.py` | Color themes for console display | `DefaultTheme`, `AnsiColorTheme` — affects `__repr__` and `show` output |

**Configuration System**

| File Path | Relevance | Key Code Points |
|---|---|---|
| `scapy/config.py` (979 lines) | Global `conf` singleton, socket backend selection | `_set_conf_sockets` (L606), `conf.verb` (verbosity), `conf.iface` (default interface), `conf.route` (routing table), `conf.L3socket` / `conf.L2socket` |
| `scapy/consts.py` | Platform detection flags (`LINUX`, `DARWIN`, `WINDOWS`) | Used to select OS-specific backend |

**Layer Binding System**

| File Path | Relevance | Key Code Points |
|---|---|---|
| `scapy/packet.py` (binding section) | Layer binding for build/dissect | `bind_layers` (L1975), `bind_bottom_up` (L1931), `bind_top_down` (L1953) |

### 0.2.2 Integration Point Discovery

- **Routing Resolution**: `IP.route()` → `conf.route.route(dst)` → returns `(iface, output_ip, gateway_ip)` — this is what Scapy displays about routing during `send()`.
- **Layer Binding Chain**: `bind_layers(Ether, IP, type=2048)` and `bind_layers(IP, TCP, frag=0, proto=6)` — these registrations control both build-time field overloading and dissection-time next-layer guessing.
- **Neighbor / MAC Resolution**: `DestMACField.i2h()` → `conf.neighbor.resolve()` → `inet_register_l3()` → `getmacbyip()` — ARP resolution happens transparently during `build()` for the Ethernet destination MAC.
- **Socket Backend Selection**: `_set_conf_sockets()` in `scapy/config.py` — selects `L3PacketSocket` / `L2Socket` on Linux, `L3bpfSocket` / `L2bpfSocket` on BSD, etc.
- **Sniff → Dissect Pipeline**: `sniff()` → `AsyncSniffer._run()` → `socket.recv()` → `SuperSocket.recv()` → `cls(val)` (calls `Packet.__init__` with raw bytes) → `dissect()` → `do_dissect()` → `do_dissect_payload()` → `guess_payload_class()` → recursive dissection.

### 0.2.3 New File Requirements

- **CREATE**: `blitzy/documentation/scapy_0925ada48540.md` — The sole deliverable: a comprehensive markdown document answering the user's questions about Scapy's runtime behavior during packet construction, transmission, and sniffing. This file documents observations and code-grounded rationale covering the full build-send-sniff lifecycle.

No other new files are required. No existing files are to be modified.


## 0.3 Dependency Inventory

### 0.3.1 Private and Public Packages

The deliverable is a pure markdown documentation file. No new runtime or build dependencies are introduced. The following table catalogs the packages relevant to understanding and executing the Scapy runtime behavior documented in this exercise:

| Registry | Package Name | Version | Purpose |
|---|---|---|---|
| PyPI | scapy | 2.7.0dev (source tree @ commit `0925ada4`) | The library under investigation — packet construction, send, sniff |
| Python | python | ≥3.7, <4 (classifiers list up to 3.10; tox tests up to 3.11) | Runtime interpreter for Scapy |
| PyPI (optional) | ipython | (optional, any) | Enhanced interactive console; detected by `scapy/main.py` at startup |
| PyPI (optional) | cryptography | ≥2.0 | TLS layer support; gated by `conf.crypto_valid` |
| PyPI (optional) | matplotlib | (optional, any) | Visualization features; not needed for this exercise |

**Version Determination Rationale:**
- `pyproject.toml` declares `requires-python = ">=3.7, <4"` and includes classifiers for Python 3.7 through 3.10.
- `tox.ini` envlist includes `py{27,34,35,36,37,38,39,310,311,...}`, confirming Python 3.11 is the highest tested version.
- The project has zero mandatory external dependencies on Linux/BSD — it runs entirely on Python standard library plus its own code.
- `setup.py` includes a Python 2 guard but the codebase targets Python ≥ 3.7 per `pyproject.toml`.

### 0.3.2 Dependency Updates

No dependency additions, removals, or version changes are required for this task. The deliverable is a standalone markdown file that requires only the existing Scapy source tree for code reference.

No import updates, external reference updates, or configuration file changes are needed.


## 0.4 Integration Analysis

### 0.4.1 Existing Code Touchpoints

This task does not modify any existing code. However, the documentation deliverable must accurately reference the following integration chains that define Scapy's runtime behavior. These are the code touchpoints that the document will analyze and describe:

**Packet Construction Chain**

```
Ether()/IP()/TCP()
```

- `Ether.__init__()` → `Packet.__init__()` (`scapy/packet.py`, L141): Initializes field dicts, sets defaults from `fields_desc`
- `/` operator → `Packet.__div__()` (L596): Clones both sides, calls `cloneA.add_payload(cloneB)`
- `Packet.add_payload()` (L360): Sets `self.payload`, calls `payload.add_underlayer(self)`, applies `overload_fields` (e.g., `Ether.type` is overloaded to `0x0800` when IP is the payload, per `bind_top_down`)
- `bind_layers(Ether, IP, type=2048)` (`scapy/layers/inet.py`, L1101): Registers both build-time overload (`bind_top_down`) and dissection-time dispatch (`bind_bottom_up`)
- `bind_layers(IP, TCP, frag=0, proto=6)` (L1113): Same bidirectional binding for IP→TCP

**Packet Display Chain**

- `Packet.__repr__()` (L552): Iterates `fields_desc`, shows only explicitly set and overloaded fields via `f.i2repr()`, uses `conf.color_theme` for colorized output, recursively calls `repr(self.payload)`
- `Packet.show()` / `_show_or_dump()` (L1459 / L1383): Hierarchical view with `###[ LayerName ]###` headers, iterates all fields (including defaults), indents payload layers
- `Packet.show2()` (L1473): Builds the packet first (`raw(self)`), then dissects it back, so auto-computed fields (checksums, lengths) appear with their calculated values
- `Packet.summary()` / `_do_summary()` (L1642 / L1617): Calls `mysummary()` on the highest layer; each layer's `mysummary()` provides a one-line description (e.g., `TCP.mysummary()` at `scapy/layers/inet.py` L825 produces `"TCP 10.0.0.1:20 > 127.0.0.1:80 S"`)

**Packet Build (Serialization) Chain**

- `Packet.build()` (L746): `do_build()` + `build_padding()` + `build_done()`
- `Packet.do_build()` (L724): `self_build()` (serializes own fields) → `post_transforms` → `do_build_payload()` → `post_build()`
- `IP.post_build()` (`scapy/layers/inet.py`, L539): Auto-computes `ihl`, `len`, and `chksum` if they are `None`
- `TCP.post_build()` (L767): Auto-computes `dataofs` and `chksum` (using pseudo-header from `self.underlayer`)

**Packet Transmission Chain**

- `send(pkt)` (`scapy/sendrecv.py`, L422): Calls `_interface_selection()` (L616) → `pkt.route()[0]` → `IP.route()` → `conf.route.route(dst)` → returns `(iface, output_ip, gateway_ip)`. Then calls `_send()` (L396) with `lambda iface: iface.l3socket()`.
- `_send()` (L396): Opens `conf.L3socket(iface=iface)` → on Linux, this is `L3PacketSocket` (`scapy/arch/linux.py`, L587)
- `__gen_send()` (L332): Iterates packets, calls `s.send(p)`, writes `b"."` to stdout for each packet when `verbose`, prints `"\nSent %i packets." % n` after completion
- `L3PacketSocket.send()` (`scapy/arch/linux.py`, L598): Calls `x.route()[0]` to determine output interface, wraps packet in L2 header if needed via `conf.l2types`, calls `self.outs.sendto(sx, sdto)`, sets `x.sent_time`

**Packet Sniff and Dissection Chain**

- `sniff()` (`scapy/sendrecv.py`, L1308): Creates `AsyncSniffer`, calls `_run()`
- `AsyncSniffer._run()` (L1064): Opens L2 listen socket via `resolve_iface(iface).l2listen()` → on Linux, `L2ListenSocket`, calls `socket.select()` in a loop, receives via `s.recv()`
- `SuperSocket.recv()` (`scapy/supersocket.py`, L172): Calls `self.recv_raw()` → gets `(cls, val, ts)` → calls `cls(val)` which triggers `Packet.__init__` with raw bytes
- `Packet.__init__()` with `_pkt=raw_bytes` (L175–178): Calls `self.dissect(_pkt)` → `pre_dissect()` → `do_dissect()` → `post_dissect()` → `extract_padding()` → `do_dissect_payload()`
- `Packet.do_dissect()` (L1002): Iterates `fields_desc`, calls `f.getfield(self, s)` for each field to consume bytes from the raw buffer
- `Packet.do_dissect_payload()` (L1023): Calls `guess_payload_class(s)` → checks `payload_guess` list (populated by `bind_bottom_up`) → instantiates the next-layer `Packet` class with remaining bytes, which recursively dissects
- `DefaultSession.on_packet_received()` (`scapy/sessions.py`, L96): Stores packet if `store=True`, calls `prn(pkt)` callback if set, prints result if callback returns non-None

### 0.4.2 Dependency Injections

No new services or dependencies need to be injected. The documentation file is a static artifact that does not participate in Scapy's runtime dependency graph.

### 0.4.3 Database/Schema Updates

No database or schema changes are required. Scapy does not use a traditional database — its "data stores" are in-memory routing tables (`conf.route`), ARP caches (`_arp_cache` in `scapy/layers/l2.py`), and protocol binding registries (`payload_guess`, `_overload_fields`), none of which are modified by this task.


## 0.5 Technical Implementation

### 0.5.1 File-by-File Execution Plan

This task produces exactly one new file and modifies zero existing files:

- **Group 1 — Documentation Deliverable:**
  - **CREATE**: `blitzy/documentation/scapy_0925ada48540.md` — Comprehensive markdown document answering the user's questions about Scapy's runtime behavior during packet construction, transmission, and sniffing. The document must include:
    - Section on Scapy interactive console startup (what happens when `scapy` is launched)
    - Section on packet construction (layer-by-layer Ether/IP/TCP stacking, display output)
    - Section on packet transmission (routing resolution, socket selection, verbose output)
    - Section on packet sniffing (capture, dissection pipeline, layer reconstruction)
    - Summary of all runtime observations with code-grounded rationale
    - Full source code references with file paths and line numbers

- **Group 2 — Temporary Investigation Scripts (created and destroyed):**
  - Any temporary Python scripts used to exercise Scapy's runtime behavior during research will be created in `/tmp/` and removed after use. These are NOT committed to the repository.

### 0.5.2 Implementation Approach

The implementation follows a research-then-document approach:

- **Establish the investigation environment** by verifying Scapy is importable and functional in the current Python runtime, and confirming the interactive console can be started programmatically.
- **Systematically exercise each phase** of the packet lifecycle (construct → send → sniff) using temporary scripts, capturing Scapy's verbose output and behavior at each step.
- **Cross-reference all observed behaviors** with the actual source code to provide evidence-backed explanations. Every claim in the document must cite the specific file, line number, and code construct that produces the observed behavior.
- **Synthesize findings** into the final `scapy_0925ada48540.md` document organized by lifecycle phase, with a closing summary that ties all observations together.
- **Clean up** by removing any temporary scripts and verifying no repository files have been modified.

### 0.5.3 Document Content Architecture

The markdown deliverable (`scapy_0925ada48540.md`) must cover the following analytical areas, each grounded in specific source code:

**Phase 1 — Construction Analysis:**
- How `Packet.__init__()` initializes fields from `fields_desc` with defaults
- How the `/` operator (`__div__`, line 596) copies and chains layers
- How `add_payload()` triggers `overload_fields` via `bind_top_down` registrations
- How `__repr__()` displays only explicitly-set and overloaded fields
- How `show()` displays all fields including defaults
- How `show2()` serializes then re-dissects to show computed fields

**Phase 2 — Transmission Analysis:**
- How `send()` calls `IP.route()` → `conf.route.route()` for interface and gateway resolution
- How `_set_conf_sockets()` selects the platform-specific socket backend
- How `L3PacketSocket.send()` wraps L3 packets in an L2 header
- How `__gen_send()` produces dot-per-packet verbose output and the final "Sent N packets" message
- How `build()` → `post_build()` auto-computes checksums, lengths, and IHL

**Phase 3 — Sniffing Analysis:**
- How `sniff()` delegates to `AsyncSniffer._run()` which opens an L2 listen socket
- How `SuperSocket.recv()` calls `cls(raw_bytes)` triggering the dissection pipeline
- How `do_dissect()` consumes bytes field-by-field from the raw buffer
- How `guess_payload_class()` uses `payload_guess` (from `bind_bottom_up`) to select the next layer
- How the recursive dissection chain reconstructs the full Ether/IP/TCP stack from raw bytes
- How `DefaultSession.on_packet_received()` processes and optionally displays each captured packet


## 0.6 Scope Boundaries

### 0.6.1 Exhaustively In Scope

- **Documentation deliverable:**
  - `blitzy/documentation/scapy_0925ada48540.md` — the sole persistent artifact

- **Source files to be analyzed and cited** (read-only, no modifications):
  - `scapy/packet.py` — Packet class: construction, stacking, build, dissect, display
  - `scapy/fields.py` — Field type system: getfield, addfield, i2repr
  - `scapy/base_classes.py` — Metaclass machinery, Gen, SetGen
  - `scapy/layers/l2.py` — Ether class, Neighbor, MAC resolution, L2 bindings
  - `scapy/layers/inet.py` — IP, TCP, UDP, ICMP classes, bind_layers, checksums
  - `scapy/layers/all.py` — Dynamic layer loader
  - `scapy/sendrecv.py` — send, sendp, sniff, AsyncSniffer, SndRcvHandler
  - `scapy/supersocket.py` — SuperSocket base, send/recv raw
  - `scapy/sessions.py` — DefaultSession, IPSession, TCPSession
  - `scapy/route.py` — Route class, route resolution
  - `scapy/arch/linux.py` — L3PacketSocket, L2Socket, L2ListenSocket
  - `scapy/arch/__init__.py` — Platform facade
  - `scapy/config.py` — conf singleton, _set_conf_sockets
  - `scapy/consts.py` — Platform detection constants
  - `scapy/main.py` — Interactive console startup, interact()
  - `scapy/all.py` — Umbrella public API import
  - `scapy/themes.py` — Color themes for display
  - `scapy/volatile.py` — Volatile/random value generators
  - `scapy/data.py` — Protocol constants (ETH_P_ALL, etc.)
  - `scapy/compat.py` — Compatibility shims (raw, bytes_encode)
  - `pyproject.toml` — Package metadata and Python version constraints
  - `tox.ini` — Test matrix and CI configuration

- **Runtime behavior to be documented:**
  - Interactive console startup sequence
  - Packet object construction with field initialization
  - Layer stacking via `/` operator
  - Field overloading via bind_layers / bind_top_down
  - Packet display: `__repr__()`, `show()`, `show2()`, `summary()`
  - Packet serialization: `build()` → `self_build()` → `post_build()` (checksum/length computation)
  - Routing resolution: `IP.route()` → `conf.route.route()`
  - Socket backend selection: `_set_conf_sockets()`
  - L3 transmission: `L3PacketSocket.send()` with L2 wrapping
  - Verbose output: dot-per-packet, "Sent N packets."
  - Sniff socket opening: `AsyncSniffer._run()` with L2ListenSocket
  - Raw byte reception: `SuperSocket.recv()` → `recv_raw()`
  - Dissection pipeline: `dissect()` → `do_dissect()` → `do_dissect_payload()` → `guess_payload_class()`
  - Session processing: `DefaultSession.on_packet_received()`

### 0.6.2 Explicitly Out of Scope

- Modifying any existing Scapy source files
- Adding any new Python code files to the repository (only a markdown document is created)
- Performance optimization of Scapy's send/sniff functions
- Refactoring of existing code
- Non-Linux platform analysis (BSD, Windows, Solaris socket backends) — the document will focus on the Linux path as the reference implementation, noting platform abstraction exists
- TLS/SSL subsystem behavior (`scapy/layers/tls/`)
- Contrib protocol modules (`scapy/contrib/`)
- Automotive subsystem (`scapy/contrib/automotive/`)
- PCAP file I/O (offline analysis)
- Protocol fuzzing (`fuzz()`)
- Protocol automata (`scapy/automaton.py`)
- Pipe processing framework (`scapy/pipetool.py`)
- Answering machine framework (`scapy/ansmachine.py`)
- Visualization and plotting features
- IPv6 routing and protocols
- CI/CD pipeline changes
- Test suite modifications


## 0.7 Rules for Feature Addition

### 0.7.1 User-Specified Implementation Rules

The following rules are explicitly mandated by the user via the **SWE-AtlasQnA-Repo** implementation rule:

- **Deliverable Format**: Create a new markdown document named `scapy_0925ada48540.md` (matching the source branch name `scapy_0925ada48540`) that comprehensively answers the questions posed in the prompt.
- **Rationale Required**: Provide thinking and rationale behind all answers. The document must explain the "why" behind each observed behavior, not just describe the "what."
- **Code as Truth**: Do not make assumptions — base all answers on the code as the truth. Every behavioral claim must be supported by specific file paths, line numbers, and code constructs from the Scapy repository.
- **No Modifications**: Do not modify any existing files in the source repository. The codebase must remain completely untouched.
- **No Additional Code**: Do not add any other code in the source repository besides the requested markdown document. Any temporary scripts used during investigation must be created outside the repository tree (e.g., in `/tmp/`) or must be fully cleaned up afterward.
- **Placement**: Place the generated document in the `blitzy/documentation` directory in the destination repository.

### 0.7.2 Repository Convention Rules

Based on analysis of the Scapy repository conventions:

- **License Headers**: Not applicable — the deliverable is a markdown document, not a Python source file. Scapy source files use `# SPDX-License-Identifier: GPL-2.0-only` headers, but documentation files do not follow this pattern.
- **Directory Structure**: The `blitzy/documentation/` directory does not currently exist and will be created as part of the deliverable.
- **File Naming**: The markdown file name is derived from the branch name per the SWE-AtlasQnA-Repo rule: `scapy_0925ada48540.md`.
- **Content Accuracy**: Per repository conventions evident in `CONTRIBUTING.md` and the detailed docstrings throughout the codebase, technical claims should be precise, citing specific code locations rather than making broad generalizations.

### 0.7.3 Repository Cleanliness Requirement

- Any temporary Python test scripts used to exercise Scapy's runtime behavior must be created in `/tmp/` or another transient location.
- After documentation is complete, verify that `git status` shows only the new `blitzy/documentation/scapy_0925ada48540.md` file and no modifications to existing tracked files.
- The repository working tree must be clean except for the one new deliverable file.


## 0.8 References

### 0.8.1 Files and Folders Searched

The following files and folders were systematically retrieved and analyzed during context gathering to derive the conclusions in this Agent Action Plan:

**Root-level files inspected:**
- `pyproject.toml` — Package metadata, Python version constraints, build system, optional dependencies
- `tox.ini` — Test matrix (Python 3.7–3.11 + PyPy), CI environments, platform coverage
- `setup.py` — Legacy build script (referenced for VERSION generation)
- `README.md` — Project overview (confirmed via folder summary)
- `CONTRIBUTING.md` — Contributor guide (confirmed via folder summary)

**Core source files read in detail:**
- `scapy/packet.py` — Lines 1–250, 340–430, 547–650, 700–810, 990–1100, 1370–1530, 1590–1700, 1931–2020 (Packet class: init, div, add_payload, repr, show, build, dissect, summary, bind_layers)
- `scapy/sendrecv.py` — Lines 1–500, 555–730, 981–1340 (send, sendp, __gen_send, sr, sniff, AsyncSniffer, SndRcvHandler)
- `scapy/layers/l2.py` — Lines 1–330 (Ether class, Neighbor, DestMACField, MAC resolution, bind_layers)
- `scapy/layers/inet.py` — Lines 521–880, 1095–1130 (IP, TCP, UDP classes, post_build, route, mysummary, bind_layers)
- `scapy/route.py` — Full file, lines 1–219 (Route class, route resolution algorithm, cache)
- `scapy/main.py` — Full file, lines 1–715 (interact, init_session, banner, IPython detection)
- `scapy/supersocket.py` — Lines 1–260 (SuperSocket base class, send, recv, recv_raw, _recv_raw)
- `scapy/sessions.py` — Full file, lines 1–397 (DefaultSession, IPSession, TCPSession, StringBuffer)
- `scapy/arch/linux.py` — Lines 475–640 (L2Socket, L2ListenSocket, L3PacketSocket — PF_PACKET sockets)
- `scapy/config.py` — Grep analysis of socket backend selection (lines 606–686)
- `scapy/all.py` — First 60 lines (umbrella imports establishing the public API)

**Folders explored:**
- Repository root (`""`) — Full structure with summary
- `scapy/` — Core runtime package with all first-order children
- `scapy/layers/` — Protocol layer registry (48 modules + TLS subfolder)
- `scapy/arch/` — Platform abstraction layer (linux, bpf, windows, libpcap, unix, solaris)

**Tech spec sections retrieved:**
- Section 1.1 — Executive Summary (project overview, version, Python requirements, stakeholders)
- Section 2.1 — Feature Catalog (F-001 through F-020, implementation status, dependencies)

### 0.8.2 Attachments

No attachments were provided for this project. No Figma URLs, design files, or supplementary documents were supplied.

### 0.8.3 External Resources Referenced

- Scapy repository: `https://github.com/secdev/scapy` (source of truth — analyzed via local clone at commit `0925ada4`)
- Scapy documentation: `https://scapy.readthedocs.io` (referenced in `pyproject.toml` and `README.md`)
- Source branch: `scapy_0925ada48540` (determines the deliverable filename per SWE-AtlasQnA-Repo rule)


