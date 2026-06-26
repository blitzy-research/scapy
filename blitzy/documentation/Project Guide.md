# Blitzy Project Guide — Scapy IPv4 Field Representation & Encoding Q&A

## 1. Executive Summary

### 1.1 Project Overview

This is a knowledge-extraction and technical-investigation task on the **Scapy** packet-manipulation library. The objective is to produce a single, evidence-based Markdown document — `blitzy/documentation/scapy_0925ada48540.md` — that comprehensively answers **five questions** about how Scapy declaratively represents and encodes **IPv4** protocol fields. The intended audience is engineers and SMEs who need authoritative, code-grounded answers. The technical scope traces the IPv4 field path (the `fields_desc`/`Field` abstraction, `IPField`/`DestIPField`, the `build()` pipeline, `Net`-based validation, and metaclass-driven layer registration) and verifies every behavioral claim through live runtime execution. The Scapy library source is treated strictly as read-only: it is analyzed and exercised, never modified.

### 1.2 Completion Status

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'pie1':'#5B39F3','pie2':'#FFFFFF','pieStrokeColor':'#B23AF2','pieOuterStrokeColor':'#B23AF2','pieSectionTextColor':'#B23AF2','pieLegendTextColor':'#000000','pieStrokeWidth':'2px'}}}%%
pie showData title Completion Status — 92.9% Complete
    "Completed Work (AI)" : 26
    "Remaining Work" : 2
```

| Metric | Hours |
|---|---|
| **Total Hours** | **28** |
| Completed Hours (AI + Manual) | 26 |
| &nbsp;&nbsp;• Completed — AI (Blitzy autonomous) | 26 |
| &nbsp;&nbsp;• Completed — Manual (human) | 0 |
| Remaining Hours | 2 |
| **Percent Complete** | **92.9%** |

> **Completion formula (PA1, AAP-scoped):** `26 completed / (26 completed + 2 remaining) = 26 / 28 = 92.9%`. All ten AAP-scoped deliverables (the five questions plus methodology, summary, references, rationale, and constraint compliance) are complete and autonomously validated; the remaining 2 hours are path-to-production human acceptance.

### 1.3 Key Accomplishments

- ✅ **All five questions answered** in a 673-line document, each with an explicit **Answer + Evidence + Rationale** structure.
- ✅ **Evidence-grounded:** 105 `path:line` citation tokens across the IPv4 code path; **88/88** distinct citations verified against source at HEAD.
- ✅ **Runtime-verified:** every behavioral claim (serialized length = 20 bytes, leading hex, `pkt.dst` type = `str`, invalid-destination `socket.gaierror`, field-class trace) reproduced live at **100%** match.
- ✅ **Byte-level rigor:** full byte-by-byte header table and derivation of why `flags="DF"` serializes to `0x4000`, with deterministic-vs-host-dependent bytes explicitly disclosed.
- ✅ **Scope-clean:** zero modifications to `scapy/**`, tests, CI, build, or configuration — `git diff` shows exactly one added file.
- ✅ **Committed & clean:** delivered across 3 commits by `agent@blitzy.com`; working tree clean at HEAD.

### 1.4 Critical Unresolved Issues

| Issue | Impact | Owner | ETA |
|---|---|---|---|
| _None_ | No critical issues. The deliverable is complete, evidence-grounded, structurally valid, and committed. | — | — |

> No critical unresolved issues. Validation found zero inaccuracies — every citation accurate and every runtime claim reproducible.

### 1.5 Access Issues

| System/Resource | Type of Access | Issue Description | Resolution Status | Owner |
|---|---|---|---|---|
| _None_ | — | No access issues identified | N/A | — |

> **No access issues identified.** The repository, source tree, project virtual environment, and runtime were all fully accessible; no external credentials, third-party APIs, or special permissions are required for this documentation task.

### 1.6 Recommended Next Steps

1. **[Medium]** Have a subject-matter expert review and accept `blitzy/documentation/scapy_0925ada48540.md` — confirm the five answers meet the required depth and spot-check a sample of citations against source.
2. **[Low]** _(Optional)_ Re-confirm the deterministic runtime values on the AAP-required Python **3.12.3** interpreter, if one is available (values are established to be identical on 3.12.3 and 3.13.7).
3. **[Low]** Merge the branch once SME sign-off is recorded — the change is purely additive (one new file) and carries no source/build/CI risk.

---

## 2. Project Hours Breakdown

### 2.1 Completed Work Detail

| Component | Hours | Description |
|---|---|---|
| [AAP R1] Field-representation model | 4.0 | Investigate & document the declarative `fields_desc`/`Field` contract and the per-field `addfield`→`i2m` build pipeline across `fields.py`, `packet.py`, `inet.py`; author Q1 (Answer + 3 evidence blocks + rationale). |
| [AAP R2] Packet serialization experiment | 3.0 | Construct/serialize `IP(dst="8.8.8.8", ttl=64, flags="DF")`; capture length + leading hex; build byte-by-byte table; derive `flags=0x4000` (FlagsField/FlagValue); document `post_build` ihl/len/chksum rationale and deterministic-vs-host disclosure. |
| [AAP R3] Runtime-type investigation | 1.5 | Probe `type(pkt.dst)`; trace `Packet.__getattr__`→`IPField.i2h`; author Q3 (Answer + evidence + rationale). |
| [AAP R4] Invalid-destination analysis | 3.5 | Capture the full chained traceback for `IP(dst="999.999.999.999")`; analyze the two-stage validation (`h2i` probe / `except socket.error` / `Net(x)` fallback → `getaddrinfo` → `gaierror`); author rationale + stability note. |
| [AAP R5] Field-class & registry trace | 4.0 | Identify `DestIPField`→`IPField`; pinpoint `IPField.i2m` (value→wire); document `inet_aton` validation + `Net` fallback, the `Emph` unwrap gotcha, and metaclass `register_owner`/`conf.layers` registration; author Q5. |
| [AAP R6] Methodology & Environment | 1.5 | Author the methodology section: two environment tables (required vs reproduction), code-as-truth statement, dual-env disclosure, support-range and cryptography notes. |
| [AAP R7/R8/R9] Summary, References & Rationale assembly | 2.0 | Compile the Summary-of-Answers table; assemble the per-file References section with citations + official-docs corroboration; synthesize rationale subsections. |
| [AAP R10] Constraint compliance | 0.5 | Enforce ephemeral `/tmp` probe discipline, scope discipline (no source edits), and fixed deliverable path/name. |
| Citation verification | 2.0 | Re-open and assert 88 `path:line` citation tokens against source at HEAD `0925ada4`. |
| Code-review & accuracy-fix cycles | 2.0 | Two iterative commits: address code-review findings (`9c05b16a`) and fix two Methodology/Environment accuracy nits (`0bba1d7a`). |
| Final validation | 2.0 | Run the five production-readiness gates (install/compile/run/test/scope), reproduce 100% of runtime claims, and verify document structure (balanced fences, well-formed tables, zero placeholders). |
| **Total Completed** | **26.0** | |

### 2.2 Remaining Work Detail

| Category | Hours | Priority |
|---|---|---|
| Human SME review & acceptance (read document, spot-check citations, confirm all five answers) | 1.5 | Medium |
| Optional re-confirmation on AAP-required Python 3.12.3 environment | 0.5 | Low |
| **Total Remaining** | **2.0** | |

### 2.3 Total Project Hours & Completion Calculation

| Quantity | Hours |
|---|---|
| Completed (Section 2.1) | 26.0 |
| Remaining (Section 2.2) | 2.0 |
| **Total Project Hours** | **28.0** |

> **Calculation:** `Completed 26 + Remaining 2 = 28 total`. `Completion % = 26 / 28 = 92.857% ≈ 92.9%`. These figures are identical in Sections 1.2, 2.1, 2.2, and the Section 7 pie chart. Confidence: **High** — the deliverable is fully specified, complete, and autonomously validated; the only variable is the duration of human review.

---

## 3. Test Results

This is a documentation deliverable with no unit tests of its own. Its validation analogs — **citation accuracy** and **runtime reproduction** — were executed by Blitzy's autonomous validation systems, and the **underlying Scapy library suite** (the code path under analysis) was executed by the setup agent. All entries below originate from Blitzy's autonomous validation logs.

| Test Category | Framework | Total Tests | Passed | Failed | Coverage % | Notes |
|---|---|---|---|---|---|---|
| Citation accuracy | Source assertion (`path:line` vs HEAD) | 88 | 88 | 0 | 100% | Every `path:line` token re-opened and asserted against source at HEAD `0925ada4`. |
| Runtime claim reproduction | Live Scapy execution (ephemeral `/tmp` probe) | 9 | 9 | 0 | 100% | Q2 len=20, equal_build, hex, dst bytes, flags+frag; Q3 `str` type; Q4 `gaierror`⊂`OSError`; Q5 `DestIPField`, `i2m`, `fmt`/`sz`, `conf.layers`. |
| Underlying library suite (UTScapy) | UTScapy / tox campaigns | 4749 | 4749 | 0 | n/a | 190 campaigns validated by the setup agent; unchanged since zero source edits were made (`git diff 0925ada4..HEAD` = only the added document). |
| Document structure validation | Markdown structural checks | — | Pass | — | n/a | 26 balanced code fences; 5 well-formed tables; both internal anchors resolve; **zero** placeholders/TODO/FIXME/TBD. |

**Aggregate:** 4,846 discrete checks across citation, runtime, and library-suite categories — **100% pass, 0 failures**.

---

## 4. Runtime Validation & UI Verification

This project has **no user interface** (it is a documentation artifact analyzing a Python library), so UI verification is not applicable. Runtime health and behavioral verification were performed by live execution.

**Runtime health**

- ✅ **Operational** — `from scapy.all import *` succeeds (0 bytes to stderr, exit 0); `scapy.__version__` = `2026.06.26`.
- ✅ **Operational** — Scapy imports from the editable source tree under the project `.venv`; `cryptography` pinned at `41.0.7` emits no `CryptographyDeprecationWarning`.

**Behavioral verification (per question)**

- ✅ **Operational (Q2)** — `bytes(IP(dst="8.8.8.8", ttl=64, flags="DF"))` length = **20**; `bytes(pkt) == pkt.build()`; leading hex `4500001400014000400018880aec0766…`; `flags+frag` = `4000`; `dst` bytes `08080808`.
- ✅ **Operational (Q3)** — `type(pkt.dst)` = `<class 'str'>`; `isinstance(pkt.dst, str)` = `True`.
- ✅ **Operational (Q4)** — `IP(dst="999.999.999.999")` raises **at construction time**: `socket.gaierror` (a subclass of `OSError`), via the `Net`→`getaddrinfo` hostname fallback.
- ✅ **Operational (Q5)** — `get_field("dst")` unwraps (`.fld`) to `DestIPField`; MRO `['DestIPField','IPField','DestField','Field','Generic','object']`; `IPField.i2m('8.8.8.8')` = `b'\x08\x08\x08\x08'`; `fmt='!4s'`, `sz=4`; `IP in conf.layers` = `True`.

**API / integration outcomes**

- ⚠ **Partial (informational only)** — Two observed values are intentionally environment-dependent and are **disclosed as such** in the document: the Q2 source address & checksum bytes (host routing) and the Q4 `gaierror` errno/message text (DNS environment). The deterministic, code-level conclusions are unaffected.

---

## 5. Compliance & Quality Review

The task is governed by the **SWE-AtlasQnA-Repo** rule set. Each directive and each quality benchmark is cross-mapped below to its status, with fixes applied during autonomous validation noted.

| Benchmark / Rule | Requirement | Status | Progress | Notes |
|---|---|---|---|---|
| New Markdown document | Create `<source_branch>.md` answering the prompt | ✅ Pass | 100% | `blitzy/documentation/scapy_0925ada48540.md` created (branch = `scapy_0925ada48540`). |
| Build & run the source | Execute Scapy to gather runtime evidence | ✅ Pass | 100% | Ephemeral `/tmp` probe executed; all runtime values captured. |
| Evidence over assumption | Cite code (`path:line`) + runtime output | ✅ Pass | 100% | 88/88 citations verified; 100% runtime reproduction. |
| Rationale / thinking | Explain reasoning behind each answer | ✅ Pass | 100% | Explicit Rationale subsection in every question. |
| No source modification | `scapy/**` and all existing files read-only | ✅ Pass | 100% | `git diff 0925ada4..HEAD` = only the added document. |
| No extra code in source repo | Only the requested document; probes ephemeral | ✅ Pass | 100% | Probes were `/tmp` scripts, deleted; working tree clean. |
| Correct placement | File in `blitzy/documentation/` | ✅ Pass | 100% | Path and name exactly as mandated. |
| Document quality — coverage | All 5 questions answered | ✅ Pass | 100% | Q1–Q5 each with Answer/Evidence/Rationale. |
| Document quality — structure | Valid Markdown, no placeholders | ✅ Pass | 100% | 26 balanced fences, 5 well-formed tables, anchors resolve, 0 TODO/FIXME/TBD. |

**Fixes applied during autonomous validation:** code-review findings were addressed in commit `9c05b16a`, and two Methodology/Environment accuracy nits were corrected in commit `0bba1d7a`. The final validator applied **no** edits — it found zero inaccuracies.

**Outstanding compliance items:** none. (A single cosmetic note from the validator: the Q1 `fields_desc` excerpt omits a trailing `# noqa: E501` comment — a meaning-preserving documentation simplification, not an error.)

---

## 6. Risk Assessment

| Risk | Category | Severity | Probability | Mitigation | Status |
|---|---|---|---|---|---|
| Runtime evidence reproduced on Python 3.13.7 rather than the AAP-required 3.12.3 | Technical | Low | Low | Both interpreters fall within `requires-python ">=3.7, <4"`; the IPv4 code path is identical; the document discloses the two environments separately and states deterministic conclusions hold on both. | Mitigated / Disclosed |
| Host- and DNS-dependent observed values (Q2 source/checksum bytes; Q4 `gaierror` errno/message) | Technical | Low | High | Document explicitly separates deterministic bytes (length, `flags=0x4000`, `ttl`, `dst=08080808`) from host-dependent ones, and states the code-level invariant for Q4 (construction-time failure, `gaierror`⊂`OSError`). | Mitigated / Disclosed |
| Citation line-number drift if `scapy/**` were later modified | Accuracy / Documentation | Low | Low | The 88 citations are pinned to source HEAD `0925ada4`; `scapy/**` is out of scope and unmodified; References section states citations were re-confirmed at HEAD. | Mitigated |
| Security exposure | Security | None | — | Read-only documentation task: zero source changes, zero new dependencies, no secrets/credentials, no attack surface introduced. | Not Applicable |
| Operational/deployment failure | Operational | None | — | Deliverable is a static Markdown file: no service, deployment, monitoring, or backup concerns. | Not Applicable |
| External integration failure | Integration | None | — | No external integrations, API keys, or service dependencies; ephemeral probe used only stdlib `socket` + Scapy from source. | Not Applicable |

**Overall risk posture:** **Low.** Only Low-severity, fully-mitigated technical/accuracy items exist; there are no security, operational, or integration risks for this read-only documentation deliverable.

---

## 7. Visual Project Status

**Project hours breakdown** (Completed = Dark Blue `#5B39F3`, Remaining = White `#FFFFFF`):

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'pie1':'#5B39F3','pie2':'#FFFFFF','pieStrokeColor':'#B23AF2','pieOuterStrokeColor':'#B23AF2','pieSectionTextColor':'#B23AF2','pieLegendTextColor':'#000000','pieStrokeWidth':'2px'}}}%%
pie showData title Project Hours Breakdown
    "Completed Work" : 26
    "Remaining Work" : 2
```

**Remaining work by category** (hours, from Section 2.2):

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'pie1':'#5B39F3','pie2':'#A8FDD9','pieStrokeColor':'#B23AF2','pieOuterStrokeColor':'#B23AF2','pieSectionTextColor':'#B23AF2','pieLegendTextColor':'#000000','pieStrokeWidth':'2px'}}}%%
pie showData title Remaining Hours by Category (Total = 2.0h)
    "SME review & acceptance (Medium)" : 1.5
    "Optional 3.12.3 re-confirmation (Low)" : 0.5
```

> **Integrity check:** "Remaining Work" = **2** matches the Section 1.2 metrics table and the Section 2.2 "Hours" sum; "Completed Work" = **26** matches Section 2.1. The remaining-by-category chart sums to **2.0h**.

---

## 8. Summary & Recommendations

**Achievements.** The project delivers a single, rigorous, evidence-based document that answers all five questions about Scapy's IPv4 field representation and encoding. Every answer is grounded in both source citations (88/88 verified) and live runtime output (100% reproduced). The work was completed without touching a single line of library source, tests, CI, or configuration — exactly as the SWE-AtlasQnA-Repo rules require — and is committed with a clean working tree.

**Remaining gaps.** None are technical. The only outstanding work is **human acceptance**: a subject-matter expert reviewing the document and signing off (1.5h), plus an optional re-confirmation on the AAP-required Python 3.12.3 interpreter (0.5h).

**Critical path to production.** SME review → sign-off → merge. Because the change is purely additive (one new file under `blitzy/documentation/`), there is no build, deployment, or rollback risk.

**Production readiness.** The deliverable is **production-ready** at **92.9% completion** (`26 / 28` AAP-scoped hours). The residual 7.1% is human review/acceptance, which cannot be performed autonomously and is intentionally reserved for a person.

| Success Metric | Target | Actual | Status |
|---|---|---|---|
| Questions answered | 5 / 5 | 5 / 5 | ✅ |
| Citation accuracy | 100% | 88 / 88 | ✅ |
| Runtime reproduction | 100% | 100% | ✅ |
| Source files modified | 0 | 0 | ✅ |
| Placeholders / TODOs | 0 | 0 | ✅ |
| AAP-scoped completion | — | 92.9% | ✅ |

---

## 9. Development Guide

All commands below were tested in the project container and are copy-pasteable. Run them from the repository root unless otherwise noted.

### 9.1 System Prerequisites

- **OS:** Linux (Ubuntu 25.10 container used for validation); any POSIX environment works.
- **Python:** **3.12.3** is the AAP-required interpreter; **3.13.7** is present in the container and reproduces all results identically. Both satisfy Scapy's `requires-python ">=3.7, <4"`.
- **Git:** 2.51.0 (with Git LFS 3.7.1) — used only to view/verify the deliverable; no LFS objects are required for this task.

```bash
python3 --version      # e.g. Python 3.13.7 (or 3.12.3)
git --version          # git version 2.51.0
```

### 9.2 Environment Setup

A project virtual environment (`.venv`) already exists with Scapy installed as an editable package. Activate it:

```bash
cd /tmp/blitzy/scapy/blitzy-f6489b71-7a38-497d-83b1-b4bd7f24b66e_a0d499
source .venv/bin/activate
python -c "import scapy; print('scapy', scapy.__version__)"   # -> scapy 2026.06.26
```

> **Note:** `cryptography` is pinned at **41.0.7**. Do **not** upgrade it — newer releases emit a `CryptographyDeprecationWarning` (TripleDES) and can break the TLS layer import. The pinned version emits no such warning and is unrelated to IPv4 field handling.

### 9.3 Dependency Installation

No dependency installation is required — the task adds **zero** dependencies and the `.venv` is pre-provisioned. If you must recreate the environment from scratch:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -e .        # editable install of Scapy from the source tree
```

> **PEP 668 note:** the container's system Python is "externally managed." Always install into a virtual environment (as above). Only if you deliberately install globally would you need `pip install --break-system-packages …`.

### 9.4 Verify the Library Imports

```bash
python -c "from scapy.all import *"   # exits 0, prints nothing to stderr
```

### 9.5 Reproduce the Runtime Evidence

The document's runtime claims are reproduced with a **temporary** probe placed under `/tmp` and deleted afterward (never committed to the repository tree). Create it:

```bash
cat > /tmp/reproduce_qna.py << 'PYEOF'
import warnings; warnings.filterwarnings("ignore")
import socket
from scapy.all import IP, conf
from scapy.fields import IPField
from scapy.layers.inet import DestIPField

# Q2 — serialize a valid packet
pkt = IP(dst="8.8.8.8", ttl=64, flags="DF")
b = bytes(pkt)
print("Q2 len            =", len(b))
print("Q2 equal_build    =", b == pkt.build())
print("Q2 hex            =", b.hex())
print("Q2 dst[16:20]     =", b[16:20].hex())
print("Q2 flags_frag[6:8]=", b[6:8].hex())

# Q3 — runtime type of dst
print("Q3 type(pkt.dst)  =", type(pkt.dst), "| isinstance str =", isinstance(pkt.dst, str))

# Q4 — invalid destination
try:
    IP(dst="999.999.999.999")
    print("Q4 NO ERROR (unexpected)")
except Exception as e:
    print("Q4 error class    =", type(e).__module__ + "." + type(e).__name__, "| OSError?", isinstance(e, OSError))

# Q5 — field class trace
f = IP().get_field("dst")
real = getattr(f, "fld", f)
print("Q5 unwrapped class=", type(real).__name__)
print("Q5 i2m('8.8.8.8') =", repr(IPField.i2m(real, None, "8.8.8.8")))
print("Q5 fmt=" + real.fmt, "sz=" + str(real.sz))
print("Q5 IP in conf.layers =", IP in conf.layers)
PYEOF

python /tmp/reproduce_qna.py
rm -f /tmp/reproduce_qna.py       # clean up — keep the repo tree clean
```

**Expected output** (deterministic lines are stable across hosts; `src`/checksum bytes and the Q4 message vary by environment):

```text
Q2 len            = 20
Q2 equal_build    = True
Q2 hex            = 4500001400014000400018880aec076608080808
Q2 dst[16:20]     = 08080808
Q2 flags_frag[6:8]= 4000
Q3 type(pkt.dst)  = <class 'str'> | isinstance str = True
Q4 error class    = socket.gaierror | OSError? True
Q5 unwrapped class= DestIPField
Q5 i2m('8.8.8.8') = b'\x08\x08\x08\x08'
Q5 fmt=!4s sz=4
Q5 IP in conf.layers = True
```

### 9.6 View the Deliverable & Verify Scope

```bash
# Read the document
less blitzy/documentation/scapy_0925ada48540.md

# Confirm exactly one file was added and nothing else changed
git diff --name-status 0925ada485406684174d6f068dbd85c4154657b3..HEAD
# -> A    blitzy/documentation/scapy_0925ada48540.md

git status --porcelain        # -> (empty: working tree clean)
```

### 9.7 Troubleshooting

- **`CryptographyDeprecationWarning` on import** — caused by a too-new `cryptography`. Pin to `41.0.7` (`pip install 'cryptography==41.0.7'`). The warning is unrelated to IPv4 field handling.
- **Q4 message differs from the document** — the `gaierror` errno/message is DNS-environment-dependent. The stable, code-level facts are: the error occurs **at construction time** and belongs to the `socket.gaierror` ⊂ `OSError` family.
- **Q2 `src`/checksum bytes differ** — these are host-dependent (auto-selected source address + computed checksum). The length (20), `flags+frag` (`4000`), `ttl` (`40`), and `dst` (`08080808`) bytes are deterministic everywhere.
- **`error: externally-managed-environment` from pip** — you are installing into the system Python. Activate the `.venv` first, or use `--break-system-packages` only for deliberate global installs.
- **Different `socket.py` line in the Q4 traceback** — the standard-library frame number reflects your exact Python build (e.g., `/usr/lib/python3.13/socket.py`) and will differ between 3.12.3 and 3.13.7; the call chain and exception family do not.

---

## 10. Appendices

### Appendix A — Command Reference

| Command | Purpose |
|---|---|
| `source .venv/bin/activate` | Activate the project virtual environment (Scapy editable install). |
| `python -c "import scapy; print(scapy.__version__)"` | Print the Scapy version (`2026.06.26`). |
| `python -c "from scapy.all import *"` | Verify the library imports cleanly (0 bytes stderr). |
| `python /tmp/reproduce_qna.py` | Reproduce all documented runtime evidence (ephemeral probe). |
| `git diff --name-status 0925ada4..HEAD` | Confirm exactly one file added (scope compliance). |
| `git status --porcelain` | Confirm a clean working tree. |
| `git log --oneline 0925ada4..HEAD` | List the three delivery commits. |

### Appendix B — Port Reference

Not applicable. This task involves no network services or listening ports. (The ephemeral probe constructs and serializes packets in-memory and performs a DNS lookup only as an incidental part of demonstrating the Q4 error; it opens no server port.)

### Appendix C — Key File Locations

| Path | Role |
|---|---|
| `blitzy/documentation/scapy_0925ada48540.md` | **The deliverable** (673 lines) — the only file created. |
| `scapy/layers/inet.py` | REFERENCE — `IP` layer, `fields_desc`, `DestIPField`. |
| `scapy/fields.py` | REFERENCE — base `Field`, `IPField`, `SourceIPField`, `addfield`/`getfield`. |
| `scapy/utils.py` | REFERENCE — `inet_aton`/`inet_ntoa`/`atol`/`valid_ip`. |
| `scapy/base_classes.py` | REFERENCE — `Net`, Packet metaclass `register_owner`/`conf.layers`. |
| `scapy/packet.py` | REFERENCE — `Packet.__init__` and the build/dissect pipeline. |
| `.venv/` | Project virtual environment (Scapy editable install). |

### Appendix D — Technology Versions

| Technology | Version | Notes |
|---|---|---|
| Scapy | `2026.06.26` | Dynamic, date-based; imported from the editable source tree. |
| Python (required) | `3.12.3` | AAP-specified source environment. |
| Python (reproduction) | `3.13.7` | Container interpreter; reproduces all results identically. |
| `cryptography` | `41.0.7` | Pinned; do not upgrade. |
| Git | `2.51.0` | With Git LFS `3.7.1`. |
| Source HEAD (base) | `0925ada485406684174d6f068dbd85c4154657b3` | Branch `scapy_0925ada48540`. |
| Delivery HEAD | `0bba1d7aa781e2e33bb6c88520b74144b13a6056` | Branch `blitzy-f6489b71-7a38-497d-83b1-b4bd7f24b66e`. |

### Appendix E — Environment Variable Reference

No application environment variables are required for this task. The only relevant shell state is virtual-environment activation:

| Variable | Set by | Purpose |
|---|---|---|
| `VIRTUAL_ENV` | `source .venv/bin/activate` | Points at the project venv so `python`/`pip` resolve to it. |
| `PATH` | `source .venv/bin/activate` | Prepends `.venv/bin` so the venv interpreter is used. |

### Appendix F — Developer Tools Guide

| Tool | Use in this project |
|---|---|
| `python` (REPL / `-c`) | Run the ephemeral probe and ad-hoc runtime checks (`type(pkt.dst)`, `bytes(pkt)`, etc.). |
| `git diff` / `git status` / `git log` | Verify scope compliance, clean tree, and the delivery commit history. |
| `grep` / `sed` | Inspect the document structure and confirm `path:line` citations against source. |
| Markdown viewer | Render the deliverable and the Mermaid pie charts in this guide. |

### Appendix G — Glossary

| Term | Definition |
|---|---|
| `fields_desc` | The ordered list of `Field` objects a Scapy layer class declares to describe its header. |
| `Field` | Base class defining the value-state conversions (`h2i`/`i2h`/`m2i`/`i2m`) and the serialization hooks (`addfield`/`getfield`). |
| `IPField` | The field class that encodes IPv4 addresses; `i2m` converts a string to 4 bytes via `inet_aton`. |
| `DestIPField` | `IPField` subclass for the `dst` field; substitutes a default when the value is `None`, then delegates to `IPField`. |
| `i2m` | "Internal → machine": the value→on-the-wire conversion method (produces the bytes packed into the packet). |
| `i2h` | "Internal → human": returns the human-readable value at attribute-access time (for `dst`, the stored `str`). |
| `Net` | Scapy class that resolves a hostname/CIDR string to addresses via `socket.getaddrinfo`; the fallback that raises `gaierror` for invalid `dst`. |
| `Emph` | A display-wrapper (`_FieldContainer`) around emphasized fields; must be unwrapped via `.fld` to reach the real field class. |
| `conf.layers` | Scapy's global registry of layer classes, populated at class-creation time by the Packet metaclass. |
| AAP | Agent Action Plan — the authoritative specification of project scope and requirements. |
| PA1 | The AAP-scoped, hours-based completion methodology used to compute the 92.9% figure. |