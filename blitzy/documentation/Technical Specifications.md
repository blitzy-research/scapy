# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

### 0.1.1 Core Documentation Objective

Based on the provided requirements, the Blitzy platform understands that the documentation objective is to **create new documentation** that comprehensively answers the user's question about Scapy's runtime behavior during three fundamental operations: packet construction, packet transmission, and packet sniffing. The user wants an observational study of what Scapy displays and reports at each stage of the build → send → sniff lifecycle, grounded entirely in the actual source code.

**Category:** Create new documentation
**Documentation Type:** Technical Q&A / Runtime Behavior Reference

The documentation requirements, restated with enhanced clarity:

- **Packet Construction Observation** — The user wants to understand what Scapy displays as each layer is added to a multi-layer packet (Ether/IP/TCP), including how default values change due to the `bind_layers()` overload mechanism (`scapy/packet.py:1975`), what the `repr()` and `show()` outputs look like at each stacking step, and how the final composite structure appears before serialization.
- **Packet Transmission Observation** — The user wants to observe what Scapy prints during the `send()` / `sendp()` / `sr1()` process, including routing resolution output from `scapy/route.py:Route.route()`, verbose progress indicators (dots for sent packets in `scapy/sendrecv.py:380`), the "Sent N packets" confirmation (`sendrecv.py:392`), and the "Begin emission / Finished sending / Received X packets, got Y answers" messages from `SndRcvHandler` (`sendrecv.py:239-223`).
- **Packet Sniffing Observation** — The user wants to see how Scapy presents received bytes when sniffing, including how raw bytes are dissected back into layered `Packet` objects via `guess_payload_class()` and the `bind_layers()` registry (`scapy/packet.py:1062`), what the `repr()` and `show()` outputs look like for sniffed packets versus constructed packets, and how `sniffed_on` and timestamp metadata are attached.
- **Summary of Findings** — A consolidated summary comparing what Scapy reveals during building, sending, and sniffing, highlighting the key differences (e.g., `show()` vs `show2()`, `None` fields before build vs computed fields after build).

### 0.1.2 Special Instructions and Constraints

The following critical directives govern this documentation task:

- **Repository immutability** — The user explicitly states: "keep the repository unchanged and clean up anything created afterward." This means **no files may be added, modified, or deleted in the source repository**. All output must reside in the `blitzy/documentation/` directory.
- **Implementation rule** — Per the `SWE-AtlasQnA-Repo` rule: "Create a new markdown document named `<source_branch_name>.md`" — the output file must be `blitzy/documentation/scapy_0925ada48540.md`.
- **Evidence-based answers** — "Do not make assumptions, base your answers on the code as the truth." All claims about Scapy's behavior must reference specific source files and line numbers.
- **Thinking/rationale required** — "Provide thinking / rationale behind the answers." The document must explain *why* Scapy behaves the way it does, citing the code mechanisms.
- **No existing file modification** — "Do not modify any existing files in the source repository."
- **Temporary scripts permitted** — The user allows temporary scripts during testing, but they must be cleaned up. The final deliverable is the markdown document only.

### 0.1.3 Technical Interpretation

These documentation requirements translate to the following technical documentation strategy:

- To **document packet construction behavior**, we will create a new markdown file analyzing `scapy/packet.py` (the `__repr__`, `show()`, `show2()`, `summary()`, `__truediv__`/`add_payload()` methods), `scapy/layers/l2.py` (the `Ether` class definition and `bind_layers(Ether, IP, type=2048)`), and `scapy/layers/inet.py` (the `IP` and `TCP` class definitions and `bind_layers(IP, TCP, frag=0, proto=6)`).
- To **document packet sending behavior**, we will analyze `scapy/sendrecv.py` (the `send()`, `sendp()`, `sr1()`, `__gen_send()`, and `SndRcvHandler` classes), `scapy/route.py` (the `Route.route()` method), and `scapy/arch/linux.py` (the `L3PacketSocket.send()` method that performs interface resolution and raw socket transmission).
- To **document packet sniffing behavior**, we will analyze `scapy/sendrecv.py` (the `sniff()` function and `AsyncSniffer._run()` method), `scapy/supersocket.py` (the `SuperSocket.recv()` method that performs dissection), and `scapy/sessions.py` (the `DefaultSession.on_packet_received()` callback).
- To **create the summary**, we will synthesize the runtime observations captured through actual Scapy execution into a coherent narrative, citing source code evidence for every behavioral claim.

### 0.1.4 Inferred Documentation Needs

Based on code analysis, the following implicit documentation needs were identified:

- **Overload fields mechanism** — The user's question about "how the final structure looks" implicitly requires explaining how `bind_layers()` registers `overload_fields` that automatically set `Ether.type=0x0800` when IP is stacked, and `IP.proto=6` when TCP is stacked (source: `scapy/packet.py:370-373`, `scapy/layers/inet.py:1101-1114`).
- **show() vs show2() distinction** — The user asks about structure "before sending," which requires clarifying the difference between `show()` (displays `None` for auto-computed fields like checksums) and `show2()` (rebuilds from bytes first, showing computed values) (source: `scapy/packet.py:1459-1486`).
- **Route resolution during send** — The `send()` function calls `_interface_selection()` which invokes `pkt.route()`, which for IP packets calls `conf.route.route(dst)` — this routing decision is a key part of "what Scapy reports during transmission" (source: `scapy/sendrecv.py:616-631`, `scapy/layers/inet.py:559-566`).
- **Dissection vs construction display differences** — When sniffing, packets are dissected from raw bytes, so `repr()` shows ALL fields explicitly set (because `explicit=1` is set during `do_dissect()` at `scapy/packet.py:1020`), whereas constructed packets only show fields that differ from defaults.


## 0.2 Documentation Discovery and Analysis

### 0.2.1 Existing Documentation Infrastructure Assessment

Repository analysis reveals a mature Sphinx-based documentation ecosystem with broad tutorial coverage but no existing document that specifically traces Scapy's display outputs across the build → send → sniff lifecycle.

**Documentation framework:** Sphinx (`>=3.0.0`) with `sphinx_rtd_theme>=0.4.3`
**Documentation generator configuration:** `doc/scapy/conf.py`
**ReadTheDocs configuration:** `.readthedocs.yml` (Ubuntu 20.04, Python 3.9, EPUB/PDF outputs)
**Documentation build command:** `sphinx-build -W --keep-going -b html . _build/html` (from CI docs job)
**API documentation tools:** `sphinx-apidoc` via tox `apitree` environment
**Diagram tools:** Mermaid (used in tech spec), no PlantUML detected
**Hosting:** ReadTheDocs at `https://scapy.readthedocs.io`

Search patterns employed and findings:

| Pattern | Files Found | Relevance |
|---------|-------------|-----------|
| `doc/scapy/*.rst` | 14 RST manual chapters | Existing manual covers usage broadly, but not runtime display output detail |
| `doc/notebooks/*.ipynb` | 3 notebooks + TLS subfolder | `Scapy in 15 minutes.ipynb` covers construction/send/sniff at tutorial level |
| `doc/scapy/usage.rst` | 1 file (primary usage guide) | Contains `show()`, `hexdump()`, `summary()` examples but without explaining the underlying code path |
| `doc/scapy/build_dissect.rst` | 1 file | Documents how to write new layers; partial overlap with packet engine internals |
| `README.md` | 1 file | Quick-start shell example only |
| `CONTRIBUTING.md` | 1 file | Contributor guide; no runtime behavior detail |

**Key finding:** The existing `doc/scapy/usage.rst` shows example outputs for `show()`, `hexdump()`, `summary()`, and `sniff()`, but does not explain *why* those outputs appear (i.e., which code paths produce them or how `bind_layers()` triggers field overloads). The user's question specifically requires this "behind-the-scenes" analysis, which no existing document provides.

### 0.2.2 Repository Code Analysis for Documentation

The following source modules were examined to extract the information needed for the documentation:

| Module | Key Elements Examined | Documentation Value |
|--------|----------------------|---------------------|
| `scapy/packet.py` | `Packet.__repr__()` (L552), `_show_or_dump()` (L1383), `show()`/`show2()` (L1459/L1473), `summary()` (L1642), `__truediv__()` (L596), `add_payload()` (L360), `build()` (L746), `do_dissect()` (L1002), `guess_payload_class()` (L1062), `bind_layers()` (L1975), `command()` (L1662) | Core display/build/dissect mechanics |
| `scapy/sendrecv.py` | `send()` (L422), `sendp()` (L452), `__gen_send()` (L332), `SndRcvHandler.__init__()` (L114), `_sndrcv_snd()` (L232), `_process_packet()` (L270), `sniff()` (L1308), `AsyncSniffer._run()` (L1064), `tshark()` (L1417) | Send/receive/sniff output behavior |
| `scapy/layers/l2.py` | `Ether` class (L244), `Ether.mysummary()` (L262), `bind_layers(Ether, IP, type=2048)` (L695-696) | Ethernet layer defaults and binding |
| `scapy/layers/inet.py` | `IP` class (L521), `IP.post_build()` (L539), `IP.route()` (L559), `IP.mysummary()` (L612), `TCP` class (L753), `TCP.post_build()` (L767), `TCP.mysummary()` (L825), `bind_layers(Ether, IP, type=2048)` (L1101), `bind_layers(IP, TCP, frag=0, proto=6)` (L1113) | IP/TCP field definitions, auto-computation, binding |
| `scapy/route.py` | `Route.route()` (L146), `Route.__repr__()` (L47) | Route resolution and display |
| `scapy/supersocket.py` | `SuperSocket.send()` (L97), `SuperSocket.recv()` (L172), `SuperSocket.recv_raw()` (L167) | Socket-level send/receive |
| `scapy/arch/linux.py` | `L3PacketSocket.send()` (L598), `L2Socket.recv_raw()` (L555) | Linux-specific socket I/O and interface binding |
| `scapy/config.py` | `_set_conf_sockets()` (L594+), `conf.L3socket`, `conf.L2socket`, `conf.L2listen` | Platform socket selection |
| `scapy/sessions.py` | `DefaultSession.on_packet_received()` (L96) | Sniff callback and `prn` output mechanism |
| `scapy/main.py` | `interact()` (L503), banner display (L589-655) | Interactive console startup |
| `scapy/themes.py` | `DefaultTheme`, `AnsiColorTheme` | Color theme applied to `repr()` and `show()` output |

### 0.2.3 Web Search Research Conducted

No external web search was required for this task. The user's question is entirely answerable from the Scapy source code, which is the authoritative truth per the project rule: "Do not make assumptions, base your answers on the code as the truth." All behavioral claims are derived directly from code inspection and live runtime execution within the Scapy virtual environment.


## 0.3 Documentation Scope Analysis

### 0.3.1 Code-to-Documentation Mapping

The following modules require documentation coverage to answer the user's question:

- **Module: `scapy/packet.py`**
  - Public APIs: `Packet.__repr__()`, `Packet.show()`, `Packet.show2()`, `Packet.summary()`, `Packet.command()`, `Packet.__truediv__()`, `Packet.add_payload()`, `Packet.build()`, `Packet.do_build()`, `Packet.do_dissect()`, `Packet.dissect()`, `Packet.guess_payload_class()`, `bind_layers()`, `bind_bottom_up()`, `bind_top_down()`, `ls()`
  - Current documentation: `doc/scapy/usage.rst` shows examples but not code-path explanations; `doc/scapy/build_dissect.rst` covers layer authoring, not runtime display
  - Documentation needed: Detailed trace of what each display method produces and why, including how `__repr__()` only shows non-default/overloaded fields while `show()` shows all fields, and how `show2()` triggers a full `build()`→`Packet(raw())` round-trip before display

- **Module: `scapy/sendrecv.py`**
  - Public APIs: `send()`, `sendp()`, `sr()`, `sr1()`, `srp()`, `sniff()`, `AsyncSniffer`, `tshark()`, `SndRcvHandler`
  - Current documentation: `doc/scapy/usage.rst` has a "Sending packets" and "Sniffing" section with basic examples
  - Documentation needed: What verbose output each function produces (dots, "Sent N packets", "Begin emission", "Received X packets"), the `prn` callback display mechanism, and how `sniffed_on` metadata is attached

- **Module: `scapy/layers/l2.py`**
  - Key class: `Ether` (fields: `dst`, `src`, `type`)
  - Binding: `bind_layers(Ether, IP, type=2048)` at line 1101 of `inet.py`
  - Documentation needed: How `Ether.type` defaults to `0x9000` (Loopback) but gets overloaded to `0x0800` (IPv4) when IP is added as payload

- **Module: `scapy/layers/inet.py`**
  - Key classes: `IP` (fields: `version`, `ihl`, `tos`, `len`, `id`, `flags`, `frag`, `ttl`, `proto`, `chksum`, `src`, `dst`, `options`), `TCP` (fields: `sport`, `dport`, `seq`, `ack`, `dataofs`, `reserved`, `flags`, `window`, `chksum`, `urgptr`, `options`)
  - Binding: `bind_layers(IP, TCP, frag=0, proto=6)` at line 1113
  - Documentation needed: How `IP.proto` defaults to `0` (hopopt) but is overloaded to `6` (tcp) when TCP is stacked; how `IP.post_build()` auto-computes `ihl`, `len`, and `chksum`; how `TCP.post_build()` auto-computes `dataofs` and `chksum`

- **Module: `scapy/route.py`**
  - Key method: `Route.route()` returning `(iface, output_ip, gateway_ip)`
  - Documentation needed: How route resolution determines the outgoing interface and source IP when `send()` is called

- **Module: `scapy/supersocket.py` + `scapy/arch/linux.py`**
  - Key methods: `SuperSocket.send()`, `SuperSocket.recv()`, `L3PacketSocket.send()`, `L2Socket.recv_raw()`
  - Documentation needed: How the socket abstraction transforms `Packet` objects into raw bytes for transmission and raw bytes back into `Packet` objects during reception

### 0.3.2 Documentation Gap Analysis

Given the requirements and repository analysis, documentation gaps include:

- **No existing document traces the full build → send → sniff lifecycle** — The usage guide shows individual examples in isolation but does not connect them into a continuous observational narrative
- **No documentation explains `repr()` filtering logic** — The `__repr__()` method (packet.py:552-586) only shows fields that are explicitly set or overloaded, but this selective display behavior is not documented anywhere
- **No documentation contrasts show() vs show2()** — Both methods exist in the usage guide but the critical difference (show2 builds from raw bytes first, computing all auto fields) is not explained
- **No documentation maps verbose output to source code** — The "Sent N packets" and "Begin emission" messages are user-visible but not documented with code references
- **No documentation explains how `bind_layers()` affects display** — The field overload mechanism that changes `Ether.type` from `0x9000` to `0x0800` when IP is stacked is central to the user's question but undocumented at the observation level


## 0.4 Documentation Implementation Design

### 0.4.1 Documentation Structure Planning

The output document (`blitzy/documentation/scapy_0925ada48540.md`) will follow a structured Q&A format with rationale, organized as:

```
blitzy/
└── documentation/
    └── scapy_0925ada48540.md
        ├── Title and Overview
        ├── 1. Starting Scapy (Interactive Console Startup)
        │   ├── What happens at import time
        │   └── Console banner and configuration
        ├── 2. Building Packets — Layer-by-Layer Observation
        │   ├── Creating individual layers (Ether, IP, TCP)
        │   ├── Stacking with the / operator
        │   ├── How bind_layers() overloads defaults
        │   ├── show() output (pre-build)
        │   ├── show2() output (post-build)
        │   └── Other display methods (hexdump, summary, command)
        ├── 3. Sending Packets — Transmission Observation
        │   ├── Route resolution (conf.route.route())
        │   ├── send() at layer 3 — verbose output
        │   ├── sendp() at layer 2 — verbose output
        │   ├── sr1() send-receive — verbose output
        │   └── What happens during build and socket I/O
        ├── 4. Sniffing Packets — Reception Observation
        │   ├── How sniff() captures packets
        │   ├── Dissection from raw bytes
        │   ├── Display of sniffed packets
        │   ├── prn callback output
        │   └── PacketList display methods
        ├── 5. Summary — Build vs Send vs Sniff
        │   ├── Key behavioral differences
        │   ├── Field display rules
        │   └── Consolidated lifecycle diagram
        └── Source References
```

### 0.4.2 Content Generation Strategy

**Information Extraction Approach:**

- "Extract display method implementations from `scapy/packet.py` lines 552-586 (`__repr__`), 1383-1457 (`_show_or_dump`), 1642-1645 (`summary`), and 1662-1693 (`command`) to explain output formatting"
- "Extract send/receive verbose output from `scapy/sendrecv.py` lines 239-250 (`_sndrcv_snd`), 216-223 (final summary), and 379-392 (`__gen_send` dots and count)"
- "Generate runtime examples by executing Scapy in the test environment and capturing actual terminal output"
- "Create diagrams by mapping the build → send → sniff data flow from `scapy/packet.py`, `scapy/sendrecv.py`, `scapy/supersocket.py`, and `scapy/route.py`"

**Documentation Standards:**

- Markdown formatting with proper headers (`#`, `##`, `###`)
- Actual Scapy session output captured and presented in fenced code blocks
- Source code citations as inline references: `Source: scapy/packet.py:LineNumber`
- Mermaid diagrams for the build, send, and sniff workflows
- All behavioral claims grounded in specific code evidence

### 0.4.3 Diagram and Visual Strategy

Mermaid diagrams to create within the output document:

- **Layer stacking flow diagram** — showing how `Ether() / IP() / TCP()` invokes `__truediv__` → `copy()` → `add_payload()` → `overload_fields` application
- **Build lifecycle diagram** — showing the `build()` → `do_build()` → `self_build()` → `post_build()` chain that computes checksums
- **Send pathway diagram** — showing `send()` → `_interface_selection()` → `pkt.route()` → `conf.route.route()` → socket creation → `SuperSocket.send()` → kernel
- **Sniff-dissection diagram** — showing `sniff()` → `AsyncSniffer._run()` → `socket.recv()` → `SuperSocket.recv()` → `cls(val)` → `Packet.__init__(_pkt=bytes)` → `dissect()` → `guess_payload_class()`
- **Lifecycle comparison table** — summarizing field states at each phase (construction: `None`, build: computed, sniffed: all explicit)


## 0.5 Documentation File Transformation Mapping

### 0.5.1 File-by-File Documentation Plan

| Target Documentation File | Transformation | Source Code/Docs | Content/Changes |
|---------------------------|----------------|------------------|-----------------|
| `blitzy/documentation/scapy_0925ada48540.md` | CREATE | `scapy/packet.py`, `scapy/sendrecv.py`, `scapy/layers/l2.py`, `scapy/layers/inet.py`, `scapy/route.py`, `scapy/supersocket.py`, `scapy/arch/linux.py`, `scapy/sessions.py`, `scapy/main.py`, `scapy/config.py`, `scapy/themes.py` | Comprehensive Q&A document answering how Scapy behaves at runtime when building Ether/IP/TCP packets, sending them, and sniffing them, with full code citations and runtime output examples |

**Note:** Per the implementation rule `SWE-AtlasQnA-Repo`, exactly one file is created. No existing repository files are modified. The `blitzy/documentation/` directory is created as the destination.

### 0.5.2 New Documentation File Detail

```
File: blitzy/documentation/scapy_0925ada48540.md
Type: Technical Q&A / Runtime Behavior Reference
Source Code:
  - scapy/packet.py (core packet engine: build, dissect, display)
  - scapy/sendrecv.py (send, sniff, sr functions and verbose output)
  - scapy/layers/l2.py (Ether class and L2 bindings)
  - scapy/layers/inet.py (IP, TCP classes and L3/L4 bindings)
  - scapy/route.py (Route.route() for interface/gateway resolution)
  - scapy/supersocket.py (SuperSocket.send/recv for raw I/O)
  - scapy/arch/linux.py (L3PacketSocket, L2Socket Linux backends)
  - scapy/sessions.py (DefaultSession sniff callback dispatch)
  - scapy/main.py (interact() console startup)
  - scapy/config.py (conf singleton, socket backend selection)
  - scapy/themes.py (color theme for repr/show output)
Sections:
  - Overview and question restatement
  - Starting Scapy (console initialization)
  - Building packets layer by layer (Ether/IP/TCP)
  - Observing the stacked packet (repr, show, show2, hexdump, summary)
  - Sending the packet (route resolution, verbose output, socket I/O)
  - Sniffing packets (capture, dissection, display)
  - Summary of build vs send vs sniff behavior
  - Source references with file paths and line numbers
Diagrams:
  - Mermaid: Layer stacking with overload_fields
  - Mermaid: Build lifecycle (build → post_build → checksums)
  - Mermaid: Send pathway (route → socket → kernel)
  - Mermaid: Sniff-dissection pipeline (recv → dissect → guess_payload_class)
Key Citations:
  - scapy/packet.py:552-586 (__repr__), 1383-1457 (_show_or_dump),
    596-607 (__truediv__), 360-377 (add_payload), 746-756 (build),
    1002-1021 (do_dissect), 1062-1079 (guess_payload_class),
    1975-1996 (bind_layers)
  - scapy/sendrecv.py:332-393 (__gen_send), 422-448 (send),
    232-268 (_sndrcv_snd), 216-223 (verbose summary), 981-1312 (AsyncSniffer/sniff)
  - scapy/layers/l2.py:244-264 (Ether class)
  - scapy/layers/inet.py:521-616 (IP class), 753-831 (TCP class),
    1101-1114 (bind_layers calls)
  - scapy/route.py:146-197 (Route.route)
  - scapy/arch/linux.py:475-631 (L2Socket, L3PacketSocket)
```

### 0.5.3 Documentation Configuration Updates

No documentation configuration files need to be updated. The output file is placed in the `blitzy/documentation/` directory, which is a standalone deliverable directory outside the Sphinx documentation tree. The Scapy documentation infrastructure (`doc/scapy/conf.py`, `.readthedocs.yml`, `tox.ini` docs environment) remains untouched.

### 0.5.4 Cross-Documentation Dependencies

- **No shared content/includes** — The output document is self-contained and does not reference or include any other documentation file
- **No navigation links** — The `blitzy/documentation/` directory is independent of the Sphinx-based `doc/scapy/` tree
- **No TOC updates** — No `index.rst`, `mkdocs.yml`, or sidebar configuration is affected
- **Source references only** — The document references Scapy source files by path and line number for traceability but does not link to or embed them


## 0.6 Dependency Inventory

### 0.6.1 Documentation Dependencies

The following tools and packages are relevant to this documentation exercise:

| Registry | Package Name | Version | Purpose |
|----------|--------------|---------|---------|
| pip | scapy | 2.7.0 (pyproject.toml) | The target library being documented; installed in development mode for runtime inspection |
| pip | sphinx | >=3.0.0 | Existing documentation generator (not used for this task, but part of project docs infrastructure) |
| pip | sphinx_rtd_theme | >=0.4.3 | ReadTheDocs theme for existing Sphinx documentation |
| system | python | 3.11 | Highest explicitly documented supported runtime (tox.ini tests py311) |
| pip | ipython | (latest compatible) | Optional interactive shell for Scapy console; used during runtime observation |
| pip | cryptography | >=2.0 | Required for Scapy's TLS/crypto layers (imported during `scapy.all`) |

**Note:** No additional documentation tools (e.g., `mkdocs`, `docusaurus`, `mermaid-cli`) are required for this task. The output is a standalone Markdown file with inline Mermaid diagram blocks that can be rendered by any Mermaid-compatible Markdown viewer (GitHub, VS Code, etc.).

### 0.6.2 Documentation Reference Updates

No existing documentation files require link updates. The output file `blitzy/documentation/scapy_0925ada48540.md` is a standalone document that does not link to or from any existing documentation file in the repository.


## 0.7 Coverage and Quality Targets

### 0.7.1 Documentation Coverage Metrics

The user's question encompasses three distinct operational phases. Coverage targets are:

| Phase | Sub-topics | Target Coverage |
|-------|-----------|-----------------|
| **Packet Construction** | Layer creation, stacking with `/`, `bind_layers()` overloads, `repr()` output, `show()` output, `show2()` output, `summary()` output, `hexdump()` output, `command()` output, `ls()` field listing | 100% — all display methods that Scapy offers for a constructed packet must be demonstrated |
| **Packet Sending** | Route resolution, `send()` verbose dots, `send()` "Sent N packets" message, `sr1()` "Begin emission" / "Finished sending" / "Received X packets" messages, socket-level build and transmit | 100% — all verbose output lines that Scapy produces during transmission must be captured and explained |
| **Packet Sniffing** | `sniff()` capture, `AsyncSniffer` mechanism, dissection from raw bytes via `guess_payload_class()`, `repr()` of sniffed packets (all fields explicit), `show()` of sniffed packets, `prn` callback output, `PacketList.nsummary()`, `sniffed_on` attribute | 100% — all sniff-related display outputs must be documented |
| **Summary** | Field state comparison (construction vs build vs sniffed), key behavioral differences, lifecycle diagram | 100% — synthesize findings into a clear comparative summary |

**Current coverage:** 0% — no existing document covers this specific Q&A topic.
**Target coverage:** 100% of the user's stated questions, with every answer citing source code.

### 0.7.2 Documentation Quality Criteria

**Completeness requirements:**

- Every behavioral claim about Scapy's output must be backed by a source code citation (`scapy/module.py:LineNumber`)
- Every display method must have an actual runtime output example captured from the Scapy environment
- The difference between `show()` and `show2()` must be explicitly demonstrated with side-by-side output
- The `bind_layers()` overload mechanism must be traced from registration (`inet.py:1101`) through application (`packet.py:370-373`) to display (`packet.py:552-586`)

**Accuracy validation:**

- All code examples shown in the document were executed in a Python 3.11 virtual environment with Scapy installed from the repository source
- Packet outputs (repr, show, hexdump) are captured verbatim from runtime execution
- Source code line references were verified against the current repository state

**Clarity standards:**

- Technical accuracy with accessible, narrative-style explanations
- Progressive disclosure: start with what the user sees, then explain why
- "Thinking / rationale" provided for every answer per the implementation rule
- Consistent Scapy terminology (layer, field, payload, underlayer, overload_fields)

**Maintainability:**

- Source file citations enable readers to navigate directly to the relevant code
- All line numbers reference the current repository commit
- Self-contained document with no external dependencies

### 0.7.3 Example and Diagram Requirements

- **Minimum examples per operational phase:** At least 3 runtime output captures (repr, show, summary) per phase
- **Diagram types required:** 4 Mermaid diagrams (layer stacking, build lifecycle, send pathway, sniff-dissection)
- **Code example verification:** All examples were executed in the test environment and their output captured verbatim
- **Visual content freshness:** All outputs reflect the current Scapy codebase (version as built from repository source)


## 0.8 Scope Boundaries

### 0.8.1 Exhaustively In Scope

**New documentation files:**

- `blitzy/documentation/scapy_0925ada48540.md` — The sole output artifact, a comprehensive Q&A markdown document

**Source code analyzed (read-only, for documentation content):**

- `scapy/packet.py` — Packet construction, display, build, dissect, bind_layers
- `scapy/sendrecv.py` — send(), sendp(), sr1(), sniff(), AsyncSniffer, SndRcvHandler
- `scapy/layers/l2.py` — Ether class, L2 bindings
- `scapy/layers/inet.py` — IP class, TCP class, L3/L4 bindings, post_build auto-computation
- `scapy/route.py` — Route.route() interface/gateway resolution
- `scapy/supersocket.py` — SuperSocket.send(), SuperSocket.recv(), raw I/O
- `scapy/arch/linux.py` — L3PacketSocket.send(), L2Socket, Linux PF_PACKET sockets
- `scapy/sessions.py` — DefaultSession.on_packet_received(), prn callback
- `scapy/main.py` — interact() console startup, banner display
- `scapy/config.py` — conf singleton, socket backend selection, verbosity settings
- `scapy/themes.py` — Color themes for repr/show output
- `scapy/base_classes.py` — SetGen, Gen base classes
- `scapy/fields.py` — Field type definitions (referenced for field serialization)
- `scapy/plist.py` — PacketList, SndRcvList display methods

**Runtime behaviors documented:**

- Ether/IP/TCP layer construction and stacking with the `/` operator
- `repr()`, `show()`, `show2()`, `summary()`, `hexdump()`, `command()`, `ls()` outputs
- `send()` and `sendp()` verbose transmission output (dots, "Sent N packets")
- `sr1()` stimulus-response output ("Begin emission", "Finished sending", "Received X packets")
- Route resolution output from `conf.route.route(dst)`
- `sniff()` packet capture and dissection from raw bytes
- `prn` callback display and `PacketList.nsummary()` output
- `sniffed_on` and timestamp metadata on sniffed packets

### 0.8.2 Explicitly Out of Scope

- **Source code modifications** — No files in the Scapy repository will be added, modified, or deleted (per user instruction: "keep the repository unchanged")
- **Test file modifications** — No test files will be created or altered in the repository
- **Existing documentation updates** — The `doc/scapy/` Sphinx tree, `README.md`, and `CONTRIBUTING.md` are not modified
- **Feature additions or code refactoring** — No code changes of any kind
- **Deployment configuration changes** — `.readthedocs.yml`, `tox.ini`, `pyproject.toml` are not modified
- **IPv6, SCTP, or other protocol stacks** — Only the Ether/IP/TCP stack is documented, as specified by the user
- **Windows, BSD, or macOS platform behavior** — Only the Linux PF_PACKET socket backend is documented (runtime environment is Linux)
- **Advanced frameworks** — Automata (`scapy/automaton.py`), Pipes (`scapy/pipetool.py`), and Answering Machines (`scapy/ansmachine.py`) are out of scope
- **TLS/crypto layers** — Not relevant to the Ether/IP/TCP question
- **Automotive/contrib layers** — Not relevant to the user's question
- **Temporary test scripts** — Any scripts created during runtime observation are cleaned up and not part of the deliverable


## 0.9 Execution Parameters

### 0.9.1 Documentation-Specific Instructions

- **Documentation build command:** Not applicable — the output is a standalone Markdown file, not part of the Sphinx build
- **Documentation preview command:** Any Markdown renderer (e.g., `grip`, VS Code Markdown preview, GitHub rendering)
- **Diagram generation command:** Mermaid diagrams are embedded inline in the Markdown and rendered by compatible viewers
- **Default format:** Markdown with Mermaid diagram blocks
- **Citation requirement:** Every section must reference source files with format `Source: scapy/module.py:LineNumber`
- **Style guide:** Follow the existing Scapy documentation tone — technical, concise, with code examples. Use the `SWE-AtlasQnA-Repo` rule structure: provide thinking/rationale behind answers
- **Documentation validation:** Verify all code example outputs match actual Scapy runtime behavior

### 0.9.2 Runtime Environment for Observation

The runtime observations documented in the output file were captured under:

| Parameter | Value |
|-----------|-------|
| **Python version** | 3.11.15 |
| **Scapy version** | Built from repository source (version 2026.04.09 dev) |
| **Platform** | Linux (PF_PACKET sockets) |
| **Socket backend** | `L3PacketSocket` (layer 3), `L2Socket` (layer 2), `L2ListenSocket` (sniff) |
| **Default interface** | `ipvlan-eth1` |
| **Verbosity** | `conf.verb = 2` (default) |
| **Color theme** | `NoTheme` (non-interactive mode); `DefaultTheme` in interactive mode |


## 0.10 Rules for Documentation

The following rules are explicitly mandated by the user and the project implementation rules:

- **"Create a new markdown document named `<source_branch_name>.md`"** — The output file must be exactly `blitzy/documentation/scapy_0925ada48540.md` (branch name: `scapy_0925ada48540`)
- **"Provide thinking / rationale behind the answers"** — Every behavioral observation about Scapy must include an explanation of *why* it happens, citing the specific code mechanism
- **"Do not make assumptions, base your answers on the code as the truth"** — All claims about Scapy's runtime output must be verifiable against source code; no external documentation or hearsay
- **"Do not modify any existing files in the source repository"** — Zero changes to any file under the repository root; only the `blitzy/documentation/` directory receives output
- **"Place the generated document in the `blitzy/documentation` directory in the destination repo"** — The `blitzy/documentation/` directory must be created and the markdown file placed there
- **"Keep the repository unchanged and clean up anything created afterward"** — Any temporary scripts or pcap files created during observation must be removed; only the final markdown document remains
- **"You may use temporary scripts while testing"** — Temporary Python scripts and pcap captures are permitted during the analysis phase but must not persist in the final deliverable
- **Source code citations required** — Every technical claim must include a file path and line number reference (e.g., `scapy/packet.py:552`) to enable verification
- **Runtime output must be verbatim** — All Scapy session outputs shown in the document must be actual captures from execution, not hand-written approximations


## 0.11 References

### 0.11.1 Source Files Searched and Analyzed

The following files and folders were systematically searched and read to derive the conclusions in this Agent Action Plan:

| File Path | Purpose in Analysis |
|-----------|-------------------|
| `scapy/packet.py` | Core packet engine — `__repr__()`, `show()`, `show2()`, `summary()`, `command()`, `__truediv__()`, `add_payload()`, `build()`, `do_build()`, `do_dissect()`, `dissect()`, `guess_payload_class()`, `bind_layers()`, `bind_bottom_up()`, `bind_top_down()`, `_show_or_dump()`, `ls()` |
| `scapy/sendrecv.py` | Send/receive module — `send()`, `sendp()`, `sr()`, `sr1()`, `sniff()`, `AsyncSniffer._run()`, `SndRcvHandler.__init__()`, `_sndrcv_snd()`, `_process_packet()`, `__gen_send()`, `tshark()`, `_interface_selection()` |
| `scapy/layers/l2.py` | Ethernet layer — `Ether` class, `Ether.mysummary()`, `Ether.answers()`, `Ether.dispatch_hook()`, `bind_layers()` calls |
| `scapy/layers/inet.py` | IP/TCP/UDP/ICMP layers — `IP` class, `IP.post_build()`, `IP.route()`, `IP.mysummary()`, `TCP` class, `TCP.post_build()`, `TCP.mysummary()`, `bind_layers(Ether, IP, type=2048)`, `bind_layers(IP, TCP, frag=0, proto=6)` |
| `scapy/route.py` | Routing engine — `Route` class, `Route.route()`, `Route.__repr__()`, `Route.resync()` |
| `scapy/supersocket.py` | Socket abstraction — `SuperSocket.send()`, `SuperSocket.recv()`, `SuperSocket.recv_raw()`, `_SuperSocket_metaclass` |
| `scapy/arch/linux.py` | Linux socket backend — `L3PacketSocket.send()`, `L3PacketSocket.recv()`, `L2Socket.__init__()`, `L2Socket.send()`, `L2Socket.recv_raw()`, `L2ListenSocket` |
| `scapy/sessions.py` | Session management — `DefaultSession.__init__()`, `DefaultSession.on_packet_received()`, `DefaultSession.toPacketList()` |
| `scapy/main.py` | Interactive console — `interact()`, `_scapy_builtins()`, `init_session()`, `QUOTES`, banner formatting |
| `scapy/config.py` | Configuration — `ConfClass`, `_set_conf_sockets()`, `conf.L3socket`, `conf.L2socket`, `conf.L2listen`, `conf.verb`, `conf.color_theme` |
| `scapy/themes.py` | Themes — `DefaultTheme`, `AnsiColorTheme`, `BlackAndWhite`, color formatting for `repr`/`show` |
| `scapy/base_classes.py` | Base classes — `SetGen`, `Gen` (used by send/recv iterators) |
| `scapy/__init__.py` | Version metadata — `VERSION`, `__version__` |
| `scapy/all.py` | Public API umbrella import |
| `pyproject.toml` | Project metadata — Python requirement, optional dependencies, entry points |
| `tox.ini` | Testing matrix — Python version support (3.7-3.11) |
| `.readthedocs.yml` | Documentation hosting configuration |
| `doc/scapy/usage.rst` | Existing usage guide — searched for overlapping content |

**Folders explored:**

| Folder Path | Depth | Findings |
|-------------|-------|----------|
| (root) | Level 0 | Identified project structure: `scapy/`, `doc/`, `test/`, `.config/`, `.github/` |
| `scapy/` | Level 1 | Core runtime package with 34 modules and 7 subpackages |
| `scapy/layers/` | Level 2 | 55+ protocol layer modules; focused on `l2.py` and `inet.py` |
| `scapy/arch/` | Level 2 | Platform backends; focused on `linux.py` for the runtime environment |
| `doc/` | Level 1 | Documentation ecosystem: notebooks, Sphinx tree, syntax, vagrant_ci |
| `doc/scapy/` | Level 2 | 14 RST chapters; confirmed no existing build-send-sniff lifecycle doc |

### 0.11.2 Attachments

No attachments were provided for this project. No Figma URLs or external design references are applicable.

### 0.11.3 Tech Spec Sections Referenced

The following sections from the existing technical specification were consulted for context:

- **1.1 Executive Summary** — Project overview, version, stakeholders, and value proposition
- **4.1 HIGH-LEVEL SYSTEM WORKFLOW** — End-to-end processing architecture, primary pipeline layers
- **4.2 CORE PACKET ENGINE PROCESSES** — Packet build lifecycle, dissection lifecycle, layer stacking and protocol binding mechanics
- **8.6 Documentation Infrastructure** — ReadTheDocs configuration, Sphinx build settings, API doc generation


