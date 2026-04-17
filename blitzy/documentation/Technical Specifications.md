# Technical Specification

# 0. Agent Action Plan

## 0.1 Intent Clarification

### 0.1.1 Core Feature Objective

Based on the prompt, the Blitzy platform understands that the new feature requirement is to produce a comprehensive investigative document that analyzes, explains, and demonstrates how DNS name compression works end-to-end within the Scapy packet manipulation library. This is an onboarding-oriented, code-truth-based Q&A exercise — not a code-modification task. Specifically, the requirements are:

- **DNS name decompression during parsing**: Explain how Scapy resolves a chain of compression pointers in an incoming DNS response, starting from raw bytes and ending with a fully qualified domain name. The user wants to know where the unwinding begins in the code and what drives the recursive pointer-following logic inside `dns_get_str()` (`scapy/layers/dns.py`, line 69).

- **Wire-format label encoding**: Document how domain names are encoded into length-prefixed label sequences before compression enters the picture, specifically through the `dns_encode()` function (`scapy/layers/dns.py`, line 154) and how the encoding produces byte sequences like `\x03www\x06google\x03com\x00`.

- **0xc0 marker discrimination**: Explain how the parser distinguishes a compression pointer (top two bits set: `cur & 0xc0`) from a normal label length byte (top two bits clear), referencing the bit-mask logic at line 103 of `dns_get_str()`.

- **Loop detection and prevention**: Clarify the `processed_pointers` list mechanism (line 88) that tracks previously visited pointer targets and breaks out of the while-loop upon revisiting a pointer (line 115–117), logging a warning via `warning("DNS decompression loop detected")`.

- **Out-of-bounds and truncation handling**: Document what happens when a compression pointer references a location outside the packet buffer (the `abs(pointer) >= max_length` guard at line 96), or when data is truncated mid-label or mid-pointer (the `pointer >= max_length` check at line 108). The parser gracefully retreats by breaking from the loop and returning whatever has been accumulated so far.

- **Cross-record-boundary decompression**: Explain the `InheritOriginDNSStrPacket._orig_s` mechanism (line 270–276) that stores the full original packet bytes on each resource record during dissection, enabling `dns_get_str()` to follow pointers that reference names in earlier record sections of the same DNS packet.

- **Compression during packet building**: Analyze the `dns_compress()` function (line 184) and its internal helpers `field_gen()` and `possible_shortens()` to explain how Scapy walks through all DNS string fields, identifies shared suffixes, and replaces later occurrences with two-byte compression pointers.

- **Deterministic decompression under varying compression strategies**: Confirm whether different compression pointer placements in a DNS response all decompress to identical domain name strings, validating that the decompression output is independent of compression strategy.

### 0.1.2 Special Instructions and Constraints

- **CRITICAL: No source repository modifications**: The user explicitly states that the repository itself should remain unchanged. Temporary scripts may be used for observation but must be cleaned up afterward.
- **SWE-AtlasQnA-Repo rule**: Create a new markdown document named `scapy_0925ada48540.md` (matching the source branch name) that comprehensively answers all questions posed in the prompt.
- **Code-as-truth requirement**: All answers must be based on the actual code analysis, not assumptions. Provide thinking and rationale behind the answers.
- **Document placement**: Place the generated document in the `blitzy/documentation` directory in the destination repository.
- **No other code additions**: No files other than the requested documentation markdown should be added.

### 0.1.3 Technical Interpretation

These feature requirements translate to the following technical implementation strategy:

- To **answer the DNS decompression questions**, we will trace through `dns_get_str()` in `scapy/layers/dns.py` (lines 69–144), documenting each branch of the while-loop, the pointer-following arithmetic, the `processed_pointers` guard, and the `_fullpacket` / `_orig_s` context-switching logic.
- To **explain wire-format encoding**, we will analyze `dns_encode()` (lines 154–172) and its label-splitting logic, combined with runtime demonstrations using the installed Scapy library.
- To **document the compression algorithm**, we will dissect `dns_compress()` (lines 184–267), its `field_gen()` iterator (lines 194–209), the `possible_shortens()` suffix generator (lines 211–215), and the compression-pointer-construction arithmetic (lines 226–230).
- To **demonstrate edge-case behavior**, we will build and run temporary Python scripts that exercise loop detection, out-of-bounds pointers, truncated data, and cross-record-boundary references, then incorporate the observed behavior into the document.
- To **produce the deliverable**, we will create a single markdown file `blitzy/documentation/scapy_0925ada48540.md` containing all findings, code references, and analysis.


## 0.2 Repository Scope Discovery

### 0.2.1 Comprehensive File Analysis

The following files have been identified through systematic repository exploration as directly relevant to the DNS compression analysis. Every file was verified via `get_source_folder_contents`, `read_file`, or `bash` search commands.

**Primary DNS Implementation Files**

| File Path | Relevance | Lines | Key Functions/Classes |
|-----------|-----------|-------|-----------------------|
| `scapy/layers/dns.py` | Core DNS layer — all compression/decompression logic | 1179 | `dns_get_str()`, `dns_encode()`, `dns_compress()`, `DNSStrField`, `DNSRRField`, `DNSQRField`, `InheritOriginDNSStrPacket`, `DNS`, `DNSQR`, `DNSRR`, `DNS_am` |
| `scapy/compat.py` | Byte-manipulation helpers used by DNS codec | 160+ | `orb()`, `chb()`, `raw()`, `bytes_encode()`, `plain_str()` |
| `scapy/error.py` | Exception and logging infrastructure | 137+ | `Scapy_Exception`, `log_runtime`, `warning()` |
| `scapy/packet.py` | Base `Packet` class — dissection chain | 2553+ | `Packet.dissect()`, `Packet.do_dissect()`, `Packet.do_dissect_payload()`, `Packet.pre_dissect()` |
| `scapy/fields.py` | Field type system — `StrLenField`, `MultipleTypeField` | 3868 | `StrLenField`, `StrField`, `MultipleTypeField`, `Field.getfield()` |

**DNS Test Files**

| File Path | Relevance | Lines |
|-----------|-----------|-------|
| `test/scapy/layers/dns.uts` | DNS unit tests including compression, loop detection, edge cases | 241 |
| `test/scapy/layers/dns_dnssec.uts` | DNSSEC resource record tests (NSEC bitmap encoding) | 146 |
| `test/scapy/layers/dns_edns0.uts` | EDNS0 OPT record tests | 91 |

**DNS-Dependent Protocol Modules (Cross-references)**

| File Path | Usage Pattern |
|-----------|---------------|
| `scapy/layers/llmnr.py` | Uses `DNSQRField`, `DNSRRField`, `DNSRRCountField` for LLMNR over DNS format |
| `scapy/layers/dhcp6.py` | Uses `DNSStrField` for DHCPv6 domain name fields |
| `scapy/layers/dcerpc.py` | Uses `DNSStrField` for DCE/RPC DNS domain name fields |
| `scapy/contrib/socks.py` | Uses `DNSStrField` for SOCKS5 domain address type |
| `scapy/contrib/pfcp.py` | APN field inspired by `DNSStrField` encoding pattern |
| `scapy/contrib/gtp.py` | APN field inspired by `DNSStrField` encoding pattern |

**Configuration and Build Files**

| File Path | Relevance |
|-----------|-----------|
| `pyproject.toml` | Python version constraint (`>=3.7, <4`), build system, package metadata |
| `tox.ini` | Testing matrix (Python 3.7–3.11), test runners |
| `setup.py` | Legacy build script, version computation |

### 0.2.2 Web Search Research Conducted

No external web search was required for this task. All questions are answerable directly from the source code, which the SWE-AtlasQnA-Repo rule requires ("base your answers on the code as the truth"). The DNS compression mechanisms implemented in Scapy follow RFC 1035 Section 4.1.4 (message compression), and the code is self-documenting with respect to the user's questions.

### 0.2.3 New File Requirements

- **CREATE**: `blitzy/documentation/scapy_0925ada48540.md` — The comprehensive analysis document answering all user questions about DNS name compression in Scapy. This is the sole deliverable.

No other new files are to be created. No existing files are to be modified. No new source code, test files, or configuration files are required per the SWE-AtlasQnA-Repo rule.

### 0.2.4 Integration Point Discovery

The DNS compression machinery integrates with the broader Scapy framework at several key points:

- **Dissection entry point**: `DNS.pre_dissect()` (line 518) validates TCP-wrapped DNS messages, then `Packet.do_dissect()` (`packet.py`, line 1002) iterates through `DNS.fields_desc`, invoking `DNSQRField.getfield()` and `DNSRRField.getfield()` which call `dns_get_str()` with `_fullpacket=True`.
- **Resource record dissection**: `DNSRRField.decodeRR()` (line 354) creates each RR with `_orig_s=s` (the full DNS packet bytes) and `_orig_p=p` (current parse offset), enabling cross-boundary pointer resolution.
- **Field-level decompression**: `DNSStrField.getfield()` (line 300) calls `dns_get_str()` with the owning packet's `_orig_s` context, allowing decompression of names within record data fields (NS, CNAME, MX, SOA, SRV, RRSIG, NSEC).
- **Compression trigger**: `DNS.compress()` (line 514) delegates to `dns_compress()` (line 184) which rebuilds the full packet, then walks all DNS string fields via `field_gen()` and replaces duplicate suffixes with pointer bytes.
- **Transport layer binding**: `bind_layers()` calls at lines 1064–1071 wire DNS to UDP port 53/5353 and TCP port 53.


## 0.3 Dependency Inventory

### 0.3.1 Private and Public Packages

The following packages are relevant to the DNS compression analysis and the document-generation deliverable. All versions are sourced directly from the repository's dependency manifests (`pyproject.toml`, `tox.ini`).

| Registry | Package | Version | Purpose |
|----------|---------|---------|---------|
| PyPI | `scapy` | 2.7.0 (dynamic from `scapy.VERSION`) | Core library under analysis — contains all DNS compression logic |
| PyPI | `setuptools` | `>=62.0.0` | Build backend declared in `pyproject.toml` line 2 |
| (stdlib) | `struct` | Python 3.7+ built-in | Binary packing/unpacking in `dns_get_str()`, `dns_encode()`, `dns_compress()` |
| (stdlib) | `socket` | Python 3.7+ built-in | Address family constants in EDNS0 client subnet handling |
| (stdlib) | `warnings` | Python 3.7+ built-in | Deprecation warnings for legacy `DNSgetstr()` |
| (stdlib) | `operator` | Python 3.7+ built-in | Floor division in EDNS0 client subnet field calculation |
| (stdlib) | `logging` | Python 3.7+ built-in | `log_runtime` logger for DNS parse warnings |

**Runtime Requirement**: Python `>=3.7, <4` as specified in `pyproject.toml` line 17. The tox.ini testing matrix confirms testing across Python 3.7, 3.8, 3.9, 3.10, and 3.11.

### 0.3.2 Dependency Updates

No dependency updates are required. This task produces a read-only documentation deliverable. The DNS layer (`scapy/layers/dns.py`) depends exclusively on:

- Internal Scapy modules: `scapy.arch`, `scapy.ansmachine`, `scapy.base_classes`, `scapy.config`, `scapy.compat`, `scapy.error`, `scapy.packet`, `scapy.fields`, `scapy.sendrecv`, `scapy.pton_ntop`, `scapy.layers.inet`, `scapy.layers.inet6` (lines 16–31 of `dns.py`)
- Python standard library modules: `operator`, `socket`, `struct`, `time`, `warnings`, `typing`

No new import statements, package installations, or version changes are needed.


## 0.4 Integration Analysis

### 0.4.1 Existing Code Touchpoints

The DNS compression system is woven through several layers of the Scapy architecture. The following documents every integration point that the analysis document must cover, with precise file locations and code-level references.

**Decompression Chain (Parsing Path)**

- `scapy/layers/dns.py` — `DNS.pre_dissect()` (line 518): Validates TCP-framed DNS messages; checks minimum length (14 bytes) and message length consistency before delegating to the standard `Packet.dissect()` chain.
- `scapy/packet.py` — `Packet.do_dissect()` (line 1002): Iterates through `DNS.fields_desc`, calling each field's `getfield()` method. For DNS, this invokes `DNSQRField.getfield()` (line 375) and `DNSRRField.getfield()` (line 375) sequentially for `qd`, `an`, `ns`, `ar` sections.
- `scapy/layers/dns.py` — `DNSRRField.getfield()` (line 375): Extracts the count from the corresponding count field (`qdcount`, `ancount`, etc.), then iterates calling `dns_get_str(s, p, _fullpacket=True)` (line 387) for each record's name, followed by `decodeRR()` (line 388) for the record body.
- `scapy/layers/dns.py` — `DNSRRField.decodeRR()` (line 354): Constructs each resource record with `_orig_s=s` and `_orig_p=p` (line 360), giving each RR access to the full packet bytes for later pointer resolution within its data fields.
- `scapy/layers/dns.py` — `DNSStrField.getfield()` (line 300): Invoked for fields like `rdata` (NS, CNAME, PTR types), `exchange` (MX), `mname`/`rname` (SOA), `target` (SRV). Calls `dns_get_str(s, 0, pkt)` where `pkt._orig_s` provides the full packet context.
- `scapy/layers/dns.py` — `dns_get_str()` (line 69): The core decompression engine. Receives the byte buffer, starting pointer, optional packet reference, and `_fullpacket` flag. Returns `(name, end_index, bytes_left, had_compression)`.

**Compression Chain (Building Path)**

- `scapy/layers/dns.py` — `DNS.compress()` (line 514): User-facing method that delegates to `dns_compress()`.
- `scapy/layers/dns.py` — `dns_compress()` (line 184): Copies the DNS layer, builds it into raw bytes (`raw(dns_pkt)` at line 192), then walks all DNS string fields via `field_gen()` (line 194) and `possible_shortens()` (line 211), building a compression dictionary mapping suffixes to pointer bytes. Applies compression by calling `setfieldval()` on each field with the pointer-enhanced value (line 257).
- `scapy/layers/dns.py` — `dns_encode()` (line 154): Called during compression to encode labels into wire format and during `DNSStrField.i2m()` (line 294). The `check_built` parameter (line 164) prevents double-encoding of already-processed values.

**Cross-Module Dependencies for Analysis**

- `scapy/compat.py` — `orb()` (line 146): Converts a byte to integer, used at `dns_get_str()` line 101 and 108 to read label/pointer bytes.
- `scapy/compat.py` — `chb()` (line 140): Converts integer to single byte, used in `dns_encode()` line 169 and `dns_compress()` lines 229.
- `scapy/error.py` — `log_runtime` (line 123): Logger used for DNS parse warnings (premature end, incomplete jump token).
- `scapy/error.py` — `warning()` (line 137): Used for decompression loop detection alert at `dns_get_str()` line 116.
- `scapy/error.py` — `Scapy_Exception` (line 30): Raised when a non-fullpacket decompression is attempted without `_orig_s` context (line 128).

### 0.4.2 Record Type Dispatch Integration

The `DNSRR_DISPATCHER` dictionary (line 1012) maps DNS record type integers to specialized packet classes. Each dispatched class inherits from `InheritOriginDNSStrPacket` (via `_DNSRRdummy`), ensuring the `_orig_s` context is propagated:

| Type ID | Class | DNS String Fields |
|---------|-------|-------------------|
| 6 | `DNSRRSOA` | `rrname`, `mname`, `rname` |
| 15 | `DNSRRMX` | `rrname`, `exchange` |
| 33 | `DNSRRSRV` | `rrname`, `target` |
| 46 | `DNSRRRSIG` | `rrname`, `signersname` |
| 47 | `DNSRRNSEC` | `rrname`, `nextname` |
| 250 | `DNSRRTSIG` | `rrname`, `algo_name` |

For the generic `DNSRR` class (line 1034), the `rdata` field is a `MultipleTypeField` (line 1042) that conditionally uses `DNSStrField` for types 2 (NS), 3 (MD), 4 (MF), 5 (CNAME), and 12 (PTR), as checked at line 1053.

### 0.4.3 Compression-Aware rdlen Recalculation

When `DNSRRField.decodeRR()` detects that a DNS string field within a record used compression (`rdata_obj.compressed` is `True`, line 367), it deletes the `rdlen` field (`del rr.rdlen` at line 368). This triggers automatic recalculation of the record data length during serialization via `_DNSRRdummy.post_build()` (line 788), ensuring that re-serialized packets have correct lengths even when compression pointers are replaced with full names.


## 0.5 Technical Implementation

### 0.5.1 File-by-File Execution Plan

This task produces exactly one new file. No existing files are modified.

**Group 1 — Deliverable Document**

- **CREATE**: `blitzy/documentation/scapy_0925ada48540.md` — Comprehensive markdown document answering all DNS compression questions, structured with the following sections:
  - Wire-format label encoding (`dns_encode()`)
  - The 0xc0 compression pointer marker and bit-mask discrimination
  - End-to-end decompression walkthrough (`dns_get_str()`)
  - Cross-record-boundary resolution via `InheritOriginDNSStrPacket._orig_s`
  - Loop detection via `processed_pointers`
  - Error handling for out-of-bounds pointers and truncated data
  - Compression during packet building (`dns_compress()`)
  - Deterministic decompression under different compression strategies
  - Code-verified examples with rationale

**Group 2 — Temporary Observation Scripts (Created and Cleaned Up)**

- **CREATE/DELETE**: Temporary Python scripts to exercise and verify DNS compression behaviors at runtime. These scripts will be used to gather evidence for the document and deleted before completion. Behaviors to observe include:
  - Basic label encoding output
  - Pointer offset arithmetic
  - Loop detection triggering
  - Out-of-bounds pointer handling
  - Truncated data graceful retreat
  - Cross-record pointer resolution
  - Compression algorithm suffix matching
  - Decompression identity across compression strategies

### 0.5.2 Implementation Approach per File

**For `blitzy/documentation/scapy_0925ada48540.md`**:

- Establish the document foundation by explaining DNS wire format and the `dns_encode()` function's label-splitting algorithm: how `b"www.google.com"` becomes `b"\x03www\x06google\x03com\x00"` through the split-and-prefix logic at line 169.
- Explain compression pointer discrimination by documenting the bit-mask `cur & 0xc0` (line 103), the two-byte pointer format where bits `[15:14] = 11` signal a pointer and bits `[13:0]` hold the offset, and the offset adjustment formula `((cur & ~0xc0) << 8) + orb(s[pointer]) - 12` (line 114).
- Trace the full decompression path from `DNS` packet construction through `DNSRRField.getfield()` → `dns_get_str()` → pointer following → name assembly, documenting each code branch.
- Document the `_orig_s` mechanism by showing how `DNSRRField.decodeRR()` passes the full packet bytes to each resource record (line 360), and how `dns_get_str()` switches from the local buffer to `s_full = pkt._orig_s` when a pointer is encountered outside _fullpacket mode (lines 90–129).
- Demonstrate loop protection by tracing the `processed_pointers` list lifecycle: initialized empty (line 88), appended after each pointer follow (line 130), checked before each jump (line 115), breaking with a warning on cycle detection (lines 116–117).
- Show error handling by documenting the `abs(pointer) >= max_length` guard (line 96), the `pointer >= max_length` incomplete-jump-token check (line 108), and the log_runtime.info messages issued before graceful break.
- Analyze the compression builder by walking through `dns_compress()`: how `field_gen()` yields all compressible DNS string fields (lines 194–209), how `possible_shortens()` generates suffix candidates from right to left (lines 211–215), how the first occurrence's byte offset is recorded and subsequent occurrences are replaced with two-byte pointers (lines 217–241).
- Validate deterministic decompression by demonstrating that two packets with the same domain name — one using a pointer, the other spelled out — both decompress to the identical bytes string.

### 0.5.3 Key Code Structures to Document

The document must explain these specific code structures with precise line references:

**`dns_get_str()` control flow (lines 69–144)**:
```
while True:
  if pointer >= max_length → break (premature end)
  read cur byte → pointer++
  if cur & 0xc0 → follow pointer, check loop
  elif cur > 0 → read label of cur bytes
  else → break (null terminator)
```

**`dns_compress()` algorithm (lines 184–267)**:
```
for each DNS string field in packet:
  for each suffix of the string:
    if suffix not seen → record its byte offset
    if suffix already seen → replace with pointer, break
```

**`InheritOriginDNSStrPacket` context propagation (lines 270–276)**:
```
class InheritOriginDNSStrPacket(Packet):
  __slots__ = [..., "_orig_s", "_orig_p"]
  def __init__(self, _orig_s=None, _orig_p=None):
    self._orig_s = _orig_s  # full packet bytes
```


## 0.6 Scope Boundaries

### 0.6.1 Exhaustively In Scope

**Analysis Target Files** (to be read and analyzed for the documentation deliverable):

- `scapy/layers/dns.py` — Complete file: all functions, classes, and integration points related to DNS compression/decompression
- `scapy/compat.py` — `orb()` (line 146), `chb()` (line 140), `raw()` (line 112), `bytes_encode()` (line 121), `plain_str()` (line 132)
- `scapy/error.py` — `Scapy_Exception` (line 30), `log_runtime` (line 123), `warning()` (line 137)
- `scapy/packet.py` — `Packet.dissect()` (line 1049), `Packet.do_dissect()` (line 1002), `Packet.do_dissect_payload()` (line 1023)
- `scapy/fields.py` — `StrLenField` (line 1888), `StrField` (line 1414), `MultipleTypeField` (line 407)
- `test/scapy/layers/dns.uts` — Test cases for compression, decompression, loop detection, edge cases
- `test/scapy/layers/dns_dnssec.uts` — DNSSEC bitmap tests
- `test/scapy/layers/dns_edns0.uts` — EDNS0 record tests
- `scapy/layers/llmnr.py` — Cross-reference: LLMNR protocol reusing DNS field types
- `scapy/layers/dhcp6.py` — Cross-reference: DHCPv6 reusing `DNSStrField`
- `scapy/layers/dcerpc.py` — Cross-reference: DCE/RPC reusing `DNSStrField`
- `scapy/contrib/socks.py` — Cross-reference: SOCKS5 reusing `DNSStrField`
- `scapy/contrib/pfcp.py` — Cross-reference: PFCP APN field inspired by DNS encoding
- `scapy/contrib/gtp.py` — Cross-reference: GTP APN field inspired by DNS encoding

**Deliverable File**:

- `blitzy/documentation/scapy_0925ada48540.md` — Single output document

**Runtime Observation Scope** (temporary scripts for evidence gathering):

- DNS label encoding verification via `dns_encode()`
- Compression pointer offset arithmetic demonstration
- Loop detection behavior under crafted pointer-cycle packets
- Out-of-bounds pointer handling observation
- Truncated label/pointer graceful retreat behavior
- Cross-record-boundary decompression via `_orig_s` context
- Compression suffix-matching algorithm walkthrough
- Deterministic decompression validation across strategies

### 0.6.2 Explicitly Out of Scope

- **Modification of any existing repository files**: Per user instructions and SWE-AtlasQnA-Repo rules, no existing files may be modified
- **Addition of new source code files**: Only the documentation markdown is added
- **DNSSEC cryptographic validation logic**: While DNSSEC record types (RRSIG, DNSKEY, DS, NSEC, NSEC3) are mentioned in the DNS layer, their cryptographic verification is not part of the DNS name compression analysis
- **DNS answering machine (`DNS_am`)**: The `DNS_am` class at line 1114 implements a DNS spoof responder and is unrelated to the compression/decompression machinery
- **DNS dynamic update functions**: `dyndns_add()` (line 1075) and `dyndns_del()` (line 1094) are high-level convenience commands, not part of compression internals
- **DNS over HTTPS (DoH) or DNS over TLS (DoT)**: These transport mechanisms are not implemented in the current Scapy DNS layer
- **EDNS0 client subnet handling**: The `EDNS0ClientSubnet` class (line 643) and associated field types deal with client subnet encoding, not name compression
- **Performance optimization or refactoring**: The analysis is observational; no changes to compression algorithm efficiency are in scope
- **Other protocol layers**: TLS, SMB, Kerberos, and other protocol implementations are not part of this analysis
- **CI/CD pipeline modifications**: No changes to `.travis.yml`, `.appveyor.yml`, `.github/`, or `tox.ini`


## 0.7 Rules for Feature Addition

### 0.7.1 User-Specified Rules

The following rules are explicitly emphasized by the user and the project's implementation rule set:

- **SWE-AtlasQnA-Repo Rule** (verbatim):
  - Create a new markdown document named `<source_branch_name>.md` that comprehensively answers the question(s) posed in the prompt.
  - Build and run the source code to analyse the repository behavior as needed.
  - Do not make assumptions, base your answers on the code as the truth.
  - Provide thinking / rationale behind the answers.
  - Do not modify any existing files in the source repository.
  - Do not add any other code in the source repository (besides the above requested document).
  - Place the generated document in the `blitzy/documentation` directory in the destination repo.

- **Repository Preservation** (user-stated): "Temporary scripts may be used for observation, but the repository itself should remain unchanged and anything temporary should be cleaned up afterward."

### 0.7.2 Derived Implementation Constraints

From the above rules, the following technical constraints apply:

- **Branch name resolution**: The current branch is `scapy_0925ada48540` (confirmed via `git branch --show-current`). The deliverable document must therefore be named `scapy_0925ada48540.md`.
- **Directory creation**: The `blitzy/documentation/` directory must be created if it does not already exist.
- **No `git add` of temporary files**: Any temporary scripts used for runtime observation must be created in a transient location (e.g., `/tmp/`) and not committed.
- **Code-as-truth standard**: Every claim in the document must reference specific lines in specific source files. Phrases like "presumably" or "likely" must be avoided; instead, use "line N of file X shows that..." or "runtime execution confirms that..."
- **Comprehensive coverage**: The document must address every question in the user's prompt — decompression, encoding, pointer discrimination, loop detection, error handling, cross-boundary resolution, compression building, and deterministic decompression.
- **Rationale required**: Each answer must include thinking or rationale explaining why the code behaves as described, not just what it does.


## 0.8 References

### 0.8.1 Repository Files and Folders Searched

The following files and folders were systematically searched and analyzed to derive the conclusions in this Agent Action Plan:

**Root-Level Exploration**
- `/` (repository root) — via `get_source_folder_contents`: identified `scapy/`, `test/`, `doc/`, `.config/`, `.github/` folders and root-level configuration files
- `pyproject.toml` — read via `bash`: confirmed Python `>=3.7, <4`, `setuptools>=62.0.0`, package metadata, optional dependencies
- `tox.ini` — read via `bash`: confirmed testing matrix (Python 3.7–3.11), test dependencies
- `setup.py` — identified via `get_source_folder_contents`: legacy build script

**Core Source Files (Deep Analysis)**
- `scapy/` — via `get_source_folder_contents`: identified all 33 top-level modules and 7 subpackages
- `scapy/layers/` — via `get_source_folder_contents`: identified all 55 protocol layer modules
- `scapy/layers/dns.py` (1179 lines) — read in full via `read_file` (5 segments: lines 1–120, 120–270, 270–450, 450–650, 650–850, 850–1050, 1050–1179): analyzed `dns_get_str()`, `dns_encode()`, `dns_compress()`, `DNSStrField`, `DNSRRField`, `DNSQRField`, `InheritOriginDNSStrPacket`, `DNS`, `DNSQR`, `DNSRR`, `DNSRRMX`, `DNSRRSOA`, `DNSRRSRV`, `DNSRRTSIG`, `DNSRR_DISPATCHER`, `DNS_am`
- `scapy/compat.py` (lines 110–160) — read via `read_file`: analyzed `raw()`, `bytes_encode()`, `plain_str()`, `chb()`, `orb()`
- `scapy/error.py` — searched via `bash grep`: identified `Scapy_Exception` (line 30), `log_runtime` (line 123), `warning()` (line 137)
- `scapy/packet.py` (lines 1002–1060) — read via `read_file`: analyzed `do_dissect()`, `do_dissect_payload()`, `dissect()`, `pre_dissect()`
- `scapy/fields.py` — searched via `bash grep`: identified `StrLenField` (line 1888), `StrField` (line 1414), `MultipleTypeField` (line 407)

**Cross-Reference Files**
- `scapy/layers/llmnr.py` — read in full via `bash cat`: confirmed reuse of `DNSQRField`, `DNSRRField`, `DNSRRCountField`, `DNS_am`
- `scapy/layers/dhcp6.py` — searched via `bash grep`: confirmed `DNSStrField` usage at lines 815, 823, 917
- `scapy/layers/dcerpc.py` — searched via `bash grep`: confirmed `DNSStrField` usage at lines 264, 268, 274
- `scapy/contrib/socks.py` — searched via `bash grep`: confirmed `DNSStrField` import and usage at line 132
- `scapy/contrib/pfcp.py` — searched via `bash grep`: confirmed DNS-inspired APN encoding at line 414
- `scapy/contrib/gtp.py` — searched via `bash grep`: confirmed DNS-inspired APN encoding at line 589

**Test Files**
- `test/scapy/layers/dns.uts` (241 lines) — read in full via `read_file`: analyzed 15+ test cases covering basic DNS operations, advanced decompression, SRV records, hidden decompression args, loop detection, premature end handling, compression round-trips, MX records, TXT records, TCP malformed messages, edge cases
- `test/scapy/layers/dns_dnssec.uts` (146 lines) — read partial via `read_file`: analyzed NSEC bitmap encoding/decoding round-trip tests
- `test/scapy/layers/dns_edns0.uts` (91 lines) — read partial via `read_file`: analyzed EDNS0 TLV and OPT record tests

**Runtime Verification Scripts**
- 6 temporary Python scripts executed via `bash` to verify: label encoding, pointer offset arithmetic, 0xc0 marker detection, loop detection triggering, out-of-bounds/truncated data handling, cross-record-boundary decompression, compression suffix matching, `_is_ptr()` detection logic, `dns_encode()` check_built behavior, non-fullpacket exception, and deterministic decompression identity

### 0.8.2 Attachments

No attachments were provided for this project. No Figma URLs or external design assets were referenced.

### 0.8.3 External References

- Scapy repository: `https://github.com/secdev/scapy`
- Scapy documentation: `https://scapy.readthedocs.io`
- DNS message compression follows RFC 1035 Section 4.1.4 (referenced implicitly by the code structure but not cited directly in the source)
- Source branch: `scapy_0925ada48540` (confirmed via `git branch --show-current`)
- Head commit: `0925ada4` — "Add AUTOSAR PDUTransport/PDU with initial batch of tests (#3933)"


