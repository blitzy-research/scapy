# Blitzy Project Guide

**Project:** Code-grounded root-cause analysis — Scapy `sr/sr1/srp` cross-gateway wrong-probe matching
**Repository:** Scapy (packet-manipulation library) · Branch `blitzy-12bda217-4c4a-49ec-9214-dce8519b7496` · Base `0925ada4` · HEAD `da5a6479`
**Deliverable type:** Documentation-only (rule set *SWE-AtlasQnA-Repo*)

---

## 1. Executive Summary

### 1.1 Project Overview

This project delivers a single, code-grounded analytical document explaining the root cause — within Scapy's actual `sr()`/`sr1()`/`srp()` request/response matching algorithm — of why a received response is paired with the **wrong** sent probe when a network-path-probing tool fires ICMP probes through multiple tunnel endpoints / gateways. The target audience is Scapy users debugging cross-gateway mismatches. The artifact is a 252-line Markdown file with 81 `path:line` citations, a Mermaid algorithm diagram, and four empirically-verified worked examples. Scope is strictly additive and documentation-only: exactly one file is created and zero existing files are modified. Business impact: an authoritative, reproducible explanation plus a code-implied remedy — delivered without patching the library, per the governing rule.

### 1.2 Completion Status

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'pie1':'#5B39F3','pie2':'#FFFFFF','pieStrokeColor':'#B23AF2','pieOuterStrokeColor':'#B23AF2','pieSectionTextColor':'#B23AF2','pieTitleTextColor':'#B23AF2','pieLegendTextColor':'#B23AF2'}}}%%
pie showData title Completion Status — 89.2% Complete
    "Completed Work (AI) — 29.0h" : 29.0
    "Remaining Work — 3.5h" : 3.5
```

| Metric | Hours |
|--------|-------|
| **Total Hours** | **32.5** |
| Completed Hours (AI = 29.0 + Manual = 0.0) | 29.0 |
| Remaining Hours | 3.5 |
| **Percent Complete** | **89.2%** |

> Completion is computed per the AAP-scoped methodology: `Completed / (Completed + Remaining) = 29.0 / 32.5 = 89.2%`. All AAP authoring scope is delivered; the remaining 3.5h is routine human path-to-production. *(Integer summary fields round Remaining 3.5h → 4h half-up; the guide uses the precise 3.5h throughout.)*

### 1.3 Key Accomplishments

- ✅ Sole AAP deliverable created at the branch-named path `blitzy/documentation/scapy_0925ada48540.md` (252 lines, 27,251 bytes).
- ✅ Full mandated **10-section** structure authored (1 H1 + 10 H2 + 14 H3).
- ✅ **81 `path:line` citations** into the four read-only reference files — independently spot-checked as 100% accurate at HEAD `0925ada4` (zero line drift).
- ✅ Root cause established and proven: disabling `conf.checkIPaddr` (and `checkIPsrc`) removes the destination IP from **both** matching stages, collapsing distinct-gateway probes into one `hashret()` bucket.
- ✅ **Four worked examples (A/B/C1/C2)** reproduced deterministically by driving Scapy's real `hashret()`/`answers()` + a faithful `SndRcvHandler` loop on in-memory packets (no root, no live network). Helper bytes independently re-verified (`caa80133`, `caa80130`, `00006400`, `0000c800`).
- ✅ Identified the Example-C2 linchpin: `IPerror.answers` evaluates `test_IPdst = self.dst == other.dst` **ungated** by `conf.checkIPaddr` (`scapy/layers/inet.py:L1022`).
- ✅ Confirmed `multi_recv` is **not** a Scapy parameter (`grep` no match; passing it raises `TypeError`); documented the real `multi` knob.
- ✅ External corroboration captured (Scapy usage docs, config API, GitHub issue #383 / PR #387).
- ✅ Repository integrity preserved: zero existing files modified; working tree clean; ephemeral `/tmp` reproduction scripts removed.
- ✅ Matching-subsystem regression suite (autonomous validation): **258 passed / 0 failed**.

### 1.4 Critical Unresolved Issues

| Issue | Impact | Owner | ETA |
|-------|--------|-------|-----|
| *None — no blocking issues identified* | The single deliverable is complete, citation-accurate, empirically reproduced, and well-formed; the working tree is clean. | — | — |

### 1.5 Access Issues

| System/Resource | Type of Access | Issue Description | Resolution Status | Owner |
|-----------------|----------------|-------------------|-------------------|-------|
| *No access issues identified* | — | The task required only local repository reads and in-process Python execution; no credentials, external services, or network access were needed or used. | N/A | — |

### 1.6 Recommended Next Steps

1. **[Medium]** Have a Scapy SME review the root-cause analysis and sign off on technical accuracy (spot-verify citations, the four worked examples, and the echo-vs-error scope boundary). *(2.0h)*
2. **[Low]** Perform an editorial / readability pass and confirm the Mermaid flowchart renders in the target Markdown viewer. *(1.0h)*
3. **[Medium]** Merge the branch/PR and confirm the artifact publishes to `blitzy/documentation/scapy_0925ada48540.md`. *(0.5h)*

---

## 2. Project Hours Breakdown

### 2.1 Completed Work Detail

| Component | Hours | Description |
|-----------|-------|-------------|
| Matching call-chain & two-stage algorithm analysis | 5.0 | Traced `sr1()`→`sr()`→`sndrcv()`→`SndRcvHandler`→`_process_packet`; documented Stage-1 `hashret()` bucketing and Stage-2 first-`answers()`-in-send-order loop; identified the 81 `path:line` citations. |
| Address-gating & ICMP-discriminator code analysis | 3.5 | `IP.hashret`/`IP.answers` `conf` gating; `ICMP.hashret`/`ICMP.answers`; the ungated `IPerror.answers` `test_IPdst`; config-flag defaults. |
| Ephemeral empirical reproduction harness | 4.0 | `/tmp` scripts driving the real `hashret()`/`answers()` plus a faithful `SndRcvHandler` matching loop over in-memory packets; deterministic; removed afterward. |
| Four worked examples with verified `hashret` bytes | 3.0 | Examples A (failure), B (default protects), C1 (unique `seq`), C2 (ICMP-error path) — each verified against real Scapy functions. |
| Markdown document authoring | 5.0 | 252 lines, 10-section structure, Mermaid flowchart, code blocks, byte-decode tables. |
| External research corroboration | 1.5 | Scapy usage documentation, config API, GitHub issue #383 / PR #387. |
| QA / review iteration cycles | 3.5 | Three follow-up commits: review findings, IP-id auto-randomization aside correction (F-1), final QA findings. |
| Final validation pass (5 production-readiness gates) | 3.5 | Citation re-verification, empirical re-reproduction, baseline tests (258/0), Markdown validity, clean-tree confirmation. |
| **Total Completed** | **29.0** | |

### 2.2 Remaining Work Detail

| Category | Hours | Priority |
|----------|-------|----------|
| Human SME technical review & sign-off of the root-cause analysis | 2.0 | Medium |
| Editorial / readability review (incl. Mermaid render check) | 1.0 | Low |
| PR merge & publication to the target documentation location | 0.5 | Medium |
| **Total Remaining** | **3.5** | |

> **Integrity check:** Section 2.1 (29.0) + Section 2.2 (3.5) = **32.5** = Total Hours (Section 1.2). Section 2.2 total (3.5) = Section 1.2 Remaining = Section 7 "Remaining Work".

---

## 3. Test Results

All tests below originate from Blitzy's autonomous validation logs for this project. Because the task is documentation-only and **no source/test code was added or changed**, these are the pre-existing matching-subsystem regression tests, executed to corroborate the documented mechanisms and to confirm zero regressions.

| Test Category | Framework | Total Tests | Passed | Failed | Coverage % | Notes |
|---------------|-----------|-------------|--------|--------|------------|-------|
| Matching-subsystem regression (`sr/sr1/srp`, `hashret`/`answers`, IP/ICMP) | Scapy UTscapy | 258 | 258 | 0 | N/A (baseline regression; no new code) | Ran `test/regression.uts` excluding `netaccess`/`tcpdump`/`manufdb`; directly corroborates echo-request/echo-reply equal `hashret()` + directional `answers()`, and the `IPerror`/`ICMPerror` path behind Example C2. |
| `py_compile` of the 4 reference files | CPython `py_compile` | 4 | 4 | 0 | N/A | `scapy/sendrecv.py`, `scapy/layers/inet.py`, `scapy/config.py`, `scapy/packet.py` all compile cleanly (exit 0). |
| Worked-example reproduction (helper facts) | In-repo Scapy + Python | 4 | 4 | 0 | N/A | `strxor`→`caa80133`/`caa80130`; `struct.pack("HH",…)`→`00006400`/`0000c800` — match the document exactly. |

**Informational (out of scope):** the *full* UTscapy suite shows `FAILED=1` from a timing-sensitive **ISOTP** test. It is unrelated to the `sr/sr1` matching subsystem, predates this task, and was not introduced by this work (zero source changes). It has no bearing on the deliverable.

---

## 4. Runtime Validation & UI Verification

This is a documentation artifact; there is **no user interface**. Runtime validation exercised the in-repository Scapy that the analysis is grounded in.

- ✅ **Operational** — In-repo Scapy imports cleanly via `PYTHONPATH=.` (`scapy.VERSION = 2026.06.26`).
- ✅ **Operational** — Default `conf` flags match the source exactly: `checkIPsrc=True`, `checkIPaddr=True`, `checkIPID=False`, `checkIPinIP=True`.
- ✅ **Operational** — Worked-example bytes reproduced deterministically (Example A identical key `0100006400`; Example B distinct keys `caa801330100006400` / `caa801300100006400`).
- ✅ **Operational** — `py_compile` clean on all four reference files.
- ✅ **Operational** — `multi_recv` confirmed non-existent: `grep -rn "multi_recv" scapy/` returns no match; passing it raises `TypeError: SndRcvHandler.__init__() got an unexpected keyword argument 'multi_recv'`.
- ✅ **Operational** — Markdown structure valid: 1 H1 + 10 H2 + 14 H3; 12 balanced code fences; one well-formed Mermaid flowchart; 81 `path:line` citations.
- ⚠ **Partial** — Mermaid diagram render in the *target* Markdown viewer not yet confirmed by a human (Low risk; syntax validated).
- ◻ **N/A** — No UI / API endpoints / services to verify (documentation deliverable).

---

## 5. Compliance & Quality Review

Mapping of the *SWE-AtlasQnA-Repo* rules and AAP deliverables to outcomes.

| # | AAP / Rule Requirement | Status | Progress | Evidence |
|---|------------------------|--------|----------|----------|
| 1 | Create `<branch>.md` answering the question comprehensively | ✅ Pass | 100% | `blitzy/documentation/scapy_0925ada48540.md` (252 lines) |
| 2 | Build & run the source to analyze behavior | ✅ Pass | 100% | In-repo Scapy imported; real `hashret()`/`answers()` + `SndRcvHandler` loop executed |
| 3 | No assumptions — code-as-truth | ✅ Pass | 100% | 81 `path:line` citations; spot-checked 100% accurate at HEAD `0925ada4` |
| 4 | Provide thinking / rationale | ✅ Pass | 100% | Call chain + decision points explained; per-attempt reasoning (Section 9) |
| 5 | Do not modify existing repository files | ✅ Pass | 100% | `git diff 0925ada4..HEAD` = 1 file added, 0 modified; 4 reference files unmodified |
| 6 | Add no other code besides the document | ✅ Pass | 100% | Only the Markdown file added; `/tmp` repro scripts removed |
| 7 | Place document in `blitzy/documentation/` | ✅ Pass | 100% | Artifact path confirmed |
| 8 | 10-section document structure | ✅ Pass | 100% | Exactly 10 H2 sections present |
| 9 | Four worked examples with real bytes | ✅ Pass | 100% | Examples A/B/C1/C2 reproduced & re-verified |
| 10 | External corroboration | ✅ Pass | 100% | Section 10.3 references (Scapy docs, config API, GH #383/#387) |
| 11 | Working tree clean / ephemeral repro | ✅ Pass | 100% | `git status --porcelain` empty |
| 12 | SME technical sign-off | ◻ Pending | 0% | Human task HT-1 (Medium) |

**Fixes applied during autonomous validation:** none required — the deliverable was found accurate, code-grounded, externally corroborated, and well-formed; applying gratuitous edits would have violated the "do not modify beyond the deliverable" rule.

---

## 6. Risk Assessment

| Risk | Category | Severity | Probability | Mitigation | Status |
|------|----------|----------|-------------|------------|--------|
| Citation line-number drift if Scapy source evolves beyond HEAD `0925ada4` | Technical | Low | Medium | Document explicitly pins to HEAD `0925ada4` / `VERSION 2026.06.26` | Mitigated |
| Worked-example byte order is endianness-dependent (`struct.pack("HH")`, little-endian x86-64) | Technical | Low | Low | Document states x86-64 little-endian explicitly | Mitigated |
| No security exposure introduced | Security | None | N/A | No source/dependency/config changes; deliverable is inert Markdown | N/A |
| Pre-existing ISOTP test flake in full UTscapy suite | Operational | Low | N/A | Out of scope; unrelated to `sr/sr1`; not introduced here; cannot be fixed without forbidden source edits | Pre-existing / Accepted |
| Doc lives in `blitzy/documentation/`, not the Sphinx `doc/` tree (lower discoverability) | Operational | Low | N/A | Intentional per governing rule | By design |
| No build/CI/runtime integration; no keys/services/network | Integration | None | N/A | Self-contained Markdown artifact | N/A |
| Root-cause correctness pending human SME sign-off | Compliance | Low | Low | Already validated autonomously + externally corroborated; SME review task HT-1 | Open |

---

## 7. Visual Project Status

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'pie1':'#5B39F3','pie2':'#FFFFFF','pieStrokeColor':'#B23AF2','pieOuterStrokeColor':'#B23AF2','pieSectionTextColor':'#B23AF2','pieTitleTextColor':'#B23AF2','pieLegendTextColor':'#B23AF2'}}}%%
pie showData title Project Hours Breakdown (Total 32.5h)
    "Completed Work" : 29.0
    "Remaining Work" : 3.5
```

**Remaining hours by category (Section 2.2) — sums to 3.5h:**

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'pie1':'#5B39F3','pie2':'#B23AF2','pie3':'#A8FDD9','pieStrokeColor':'#B23AF2','pieSectionTextColor':'#000000','pieTitleTextColor':'#B23AF2','pieLegendTextColor':'#B23AF2'}}}%%
pie showData title Remaining Work by Category (3.5h)
    "SME technical review & sign-off (Medium)" : 2.0
    "Editorial / readability review (Low)" : 1.0
    "PR merge & publication (Medium)" : 0.5
```

> **Integrity:** "Remaining Work" = **3.5h** here = Section 1.2 Remaining = Section 2.2 total. "Completed Work" = **29.0h** = Section 2.1 total. Colors: Completed = Dark Blue `#5B39F3`, Remaining = White `#FFFFFF`.

---

## 8. Summary & Recommendations

**Achievements.** The project is **89.2% complete**. The sole AAP deliverable — a code-grounded root-cause analysis of Scapy's `sr/sr1/srp` cross-gateway wrong-probe matching — has been authored, validated, and committed. It contains the mandated 10-section structure, 81 `path:line` citations verified at HEAD `0925ada4`, a Mermaid algorithm diagram, and four worked examples reproduced deterministically against the real in-repository `hashret()`/`answers()`. The root cause is established conclusively: with `conf.checkIPaddr=False` (and `checkIPsrc=False`), the destination IP is removed from **both** matching stages, so distinct-gateway echo probes that share `(id, seq)` — recall ICMP `id` defaults to a fixed `0` — collapse into a single `hashret()` bucket, and the reply is paired with the earliest-sent member. The document further proves the ICMP-error/traceroute path remains correct (ungated `IPerror.answers` `test_IPdst`), debunks the non-existent `multi_recv`, explains each user remediation, and disambiguates the matcher from `IPSession`/`TCPSession` reassembly.

**Remaining gaps & critical path to production.** Only routine human path-to-production remains (3.5h): SME technical sign-off (2.0h), an editorial/readability pass (1.0h), and PR merge (0.5h). There are **no blocking issues** and no High-priority tasks.

**Success metrics.** Repository integrity preserved (1 file added, 0 modified, clean tree); citation accuracy 100%; worked-example bytes reproduced exactly; matching-subsystem regression suite 258/0.

**Production-readiness assessment.** The deliverable is **release-ready pending human review**. Given the strict documentation-only scope, full code-grounding, and clean validation, confidence is **High**.

| Metric | Value |
|--------|-------|
| AAP-scoped completion | 89.2% |
| Completed / Remaining / Total hours | 29.0 / 3.5 / 32.5 |
| AAP requirements completed | 18 / 18 (authoring scope) |
| Blocking issues | 0 |
| Files added / modified | 1 / 0 |
| Matching-suite tests (autonomous) | 258 passed / 0 failed |
| Confidence | High |

---

## 9. Development Guide

This guide reproduces and verifies the deliverable's code-grounded claims. Every command was tested. Run from the repository root.

### 9.1 System Prerequisites

- **OS:** Linux/macOS (verified on Ubuntu container).
- **Python:** 3.x. Scapy declares `requires-python = ">=3.7, <4"`; the analysis is pure-Python and version-agnostic (verified on 3.13.7; AAP used 3.12.3).
- **Git:** any recent version (verified 2.51.0).
- **Disk:** ~200 MB for the repository.
- **Privileges:** none — all verification is in-memory (no root, no live network).

### 9.2 Environment Setup

```bash
# From the repository root
cd /path/to/repo

# Activate the provided virtual environment (or create one)
source .venv/bin/activate          # python -> .venv/bin/python
# If creating fresh instead:
#   python3 -m venv .venv && source .venv/bin/activate
```

> **Note (PEP 668):** the system Python is marked *externally-managed*. Prefer the venv above; if you must install globally, append `--break-system-packages`.

### 9.3 Dependency Installation

Scapy's core has **no mandatory third-party dependencies**. Import the in-repository package directly:

```bash
PYTHONPATH=. python3 -c "import scapy; print(scapy.VERSION)"
# Expected: 2026.06.26
```

### 9.4 Verify the Code-Grounded Claims

```bash
# 1) Runtime + default conf flags (must match the document)
PYTHONPATH=. python3 -c "import scapy; from scapy.config import conf; \
print(scapy.VERSION, conf.checkIPsrc, conf.checkIPaddr, conf.checkIPID, conf.checkIPinIP)"
# Expected: 2026.06.26 True True False True

# 2) Verify the Example-C2 linchpin citation (IPerror.answers test_IPdst, ungated)
sed -n '1021,1022p' scapy/layers/inet.py
# Expected line 1022: test_IPdst = self.dst == other.dst

# 3) Reproduce worked-example helper bytes
PYTHONPATH=. python3 -c "import struct, socket; from scapy.utils import strxor; \
print('GW1 key:', '01'+struct.pack('HH',0,100).hex()); \
print('GW1 xor:', strxor(socket.inet_pton(socket.AF_INET,'192.168.1.50'), socket.inet_pton(socket.AF_INET,'10.0.0.1')).hex())"
# Expected: GW1 key: 0100006400   |   GW1 xor: caa80133

# 4) Compile-check the four read-only reference files
PYTHONPATH=. python3 -m py_compile scapy/sendrecv.py scapy/layers/inet.py scapy/config.py scapy/packet.py && echo "py_compile OK"

# 5) Validate the Markdown structure (use Python — a shell backtick grep can hang)
PYTHONPATH=. python3 - <<'PY'
import re
s = open('blitzy/documentation/scapy_0925ada48540.md', encoding='utf-8').read()
L = s.splitlines()
print('lines       =', len(L))
print('code fences =', sum(l.strip().startswith('```') for l in L), '(even => balanced)')
print('H2 sections =', sum(l.startswith('## ') for l in L), '(expected 10)')
print('citations   =', len(re.findall(r'scapy/[A-Za-z_/]+\.py:L\d+', s)))
PY
# Expected: lines 252 | fences 12 | H2 10 | citations 81
```

### 9.5 Run the Matching-Subsystem Regression Suite (optional)

```bash
PYTHONPATH=. python3 -m scapy.tools.UTscapy -t test/regression.uts -K netaccess -K tcpdump -K manufdb
# Autonomous validation result: PASSED=258 FAILED=0
```

### 9.6 Example Usage — Read the Deliverable / Reproduce the Failure

```bash
# View the document
sed -n '1,40p' blitzy/documentation/scapy_0925ada48540.md

# Conceptual reproduction:
#   Example A (conf.checkIPaddr=False, identical id=0/seq=100):
#     both gateway probes hash to identical key 0100006400 -> same bucket -> reply paired with EARLIEST-sent probe (wrong).
#   Example B (default conf.checkIPaddr=True):
#     distinct keys caa801330100006400 / caa801300100006400 -> separate buckets -> correct pairing.
```

### 9.7 Troubleshooting

- **`ImportError: cannot import name 'inet_pton' from 'scapy.compat'`** → use `socket.inet_pton(socket.AF_INET, ...)` instead (as in the commands above).
- **Shell hangs when counting code fences with backtick `grep`** → use the Python validator in §9.4 step 5.
- **`error: externally-managed-environment` from pip** → use the `.venv` (preferred) or pass `--break-system-packages`.
- **Worked-example bytes differ** → `struct.pack("HH", …)` is little-endian on x86-64; results differ on big-endian hosts (the document notes this).
- **`git status` shows changes** → never modify the four reference files; revert any inadvertent edits to keep the tree clean.

---

## 10. Appendices

### A. Command Reference

| Purpose | Command |
|---------|---------|
| Import & version | `PYTHONPATH=. python3 -c "import scapy; print(scapy.VERSION)"` |
| Dump `conf` flags | `PYTHONPATH=. python3 -c "from scapy.config import conf; print(conf.checkIPsrc, conf.checkIPaddr, conf.checkIPID, conf.checkIPinIP)"` |
| Verify a citation | `sed -n '<start>,<end>p' scapy/layers/inet.py` |
| Compile-check refs | `PYTHONPATH=. python3 -m py_compile scapy/sendrecv.py scapy/layers/inet.py scapy/config.py scapy/packet.py` |
| Matching regression suite | `PYTHONPATH=. python3 -m scapy.tools.UTscapy -t test/regression.uts -K netaccess -K tcpdump -K manufdb` |
| Confirm clean tree | `git status --porcelain` |
| Confirm diff scope | `git diff 0925ada4..HEAD --name-status` |

### B. Port Reference

Not applicable — the deliverable is a documentation artifact; no services, ports, or network listeners are involved.

### C. Key File Locations

| Path | Role |
|------|------|
| `blitzy/documentation/scapy_0925ada48540.md` | **The deliverable** (created; 252 lines) |
| `scapy/sendrecv.py` | Reference — `SndRcvHandler`, `multi`, `hsent` indexing, `_process_packet`, `sr`/`sr1`/`srp` |
| `scapy/layers/inet.py` | Reference — `IP.hashret`/`answers`, `ICMP.hashret`/`answers`, `IPerror.answers`, `icmp_id_seq_types` |
| `scapy/config.py` | Reference — `checkIPID`/`checkIPsrc`/`checkIPaddr`/`checkIPinIP` defaults |
| `scapy/packet.py` | Reference — base `hashret`/`answers`, `NoPayload.hashret` |
| `test/regression.uts` | Matching-subsystem regression tests (autonomous validation) |

### D. Technology Versions

| Component | Version | Notes |
|-----------|---------|-------|
| Scapy (in-repo) | 2026.06.26 | The library under analysis; HEAD `0925ada4` |
| CPython | 3.12.3 (AAP) / 3.13.7 (verification host) | Analysis is pure-Python / version-agnostic |
| Git | 2.51.0 | For diff/status verification |

### E. Environment Variable Reference

| Variable | Value | Purpose |
|----------|-------|---------|
| `PYTHONPATH` | `.` (repo root) | Imports the in-repository Scapy rather than any installed copy |

No application secrets, API keys, or service credentials are required.

### F. Developer Tools Guide

- **UTscapy** (`scapy/tools/UTscapy.py`) — Scapy's unit-test runner. Use `-t <file>` to select a campaign and `-K <keyword>` to exclude keyword-tagged tests (e.g., `netaccess`, `tcpdump`, `manufdb`) so the suite runs without root or network.
- **`py_compile`** — fast read-only syntax check of the reference files (no bytecode side effects committed).
- **Python Markdown validator** (§9.4 step 5) — counts lines, fences, H2 sections, and citations without relying on a shell backtick `grep`.

### G. Glossary

| Term | Meaning |
|------|---------|
| `hashret()` | Per-layer function returning the bucket key used to group sent packets; a reply that answers a probe computes the same key. |
| `answers()` | Per-layer predicate testing whether a received packet answers a given sent packet. |
| Stage 1 / Stage 2 | (1) Bucket sent packets by `hashret()`; (2) pair a reply with the **first** bucket member whose `answers()` is true, in send order. |
| `conf.checkIPaddr` | Config flag (default `True`) gating whether IP addresses enter `hashret`/`answers`; disabling it removes destination-based disambiguation. |
| `IPerror`/`ICMPerror` | Layers Scapy uses for the quoted inner packet inside ICMP error messages; `IPerror.answers` compares the quoted destination **ungated** by `conf.checkIPaddr`. |
| `multi` | The real `SndRcvHandler` parameter (default `False`) controlling whether one stimulus may collect multiple answers; **not** `multi_recv` (which does not exist). |
| `IPSession`/`TCPSession` | Scapy's fragment/stream reassembly machinery for `sniff(session=...)` — a **separate** subsystem, not involved in `sr/sr1` matching. |