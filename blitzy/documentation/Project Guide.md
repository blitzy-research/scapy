# Blitzy Project Guide — Scapy ICMP-Error ↔ Echo-Request Matching Investigation

> **Task type:** Read-only, runtime-grounded QnA-Repo documentation investigation
> **Repository:** Scapy checkout at branch `scapy_0925ada48540` (base `0925ada4`)
> **Delivery branch:** `blitzy-3f5cfafe-471a-4fd3-a88f-6526e04f5244` @ `51225ee6`
> **Sole deliverable:** `blitzy/documentation/scapy_0925ada48540.md` (1,083 lines)

---

## 1. Executive Summary

### 1.1 Project Overview

This project answers six precise technical questions about **how Scapy matches a returning ICMP *destination-unreachable* error to the original ICMP *echo-request* that elicited it**, grounding every answer in actual runtime observation rather than code-reading alone. The audience is protocol engineers and Scapy maintainers who need an authoritative, evidence-backed explanation of the request/response matching internals (the two-stage `hashret()`→`answers()` path, embedded-packet re-dissection, configuration toggles, byte-order tolerance, and RFC 4884 extensions). The technical scope is a single, self-contained markdown answer document produced under a strict read-only mandate: no Scapy source file is modified, and all observations are captured by running the real code paths through their public API.

### 1.2 Completion Status

The completion percentage is calculated using the AAP-scoped hours methodology: **Completed Hours ÷ Total Hours**. Every AAP deliverable is complete; the remaining hours represent the standard human path-to-acceptance (SME review + PR merge) for a documentation artifact.

```mermaid
pie showData title Completion Status — 87.5% Complete
    "Completed (Dark Blue #5B39F3)" : 28.0
    "Remaining (White #FFFFFF)" : 4
```

| Metric | Hours |
|---|---|
| **Total Hours** | **32** |
| Completed Hours (AI: 28.0 + Manual: 0.0) | 28.0 |
| Remaining Hours | 4 |
| **Percent Complete** | **87.5%** |

> **Formula:** 28 ÷ 32 = **87.5%** complete. Completed = **Dark Blue `#5B39F3`**; Remaining = **White `#FFFFFF`**.

### 1.3 Key Accomplishments

- ✅ **Single deliverable authored and committed** — `blitzy/documentation/scapy_0925ada48540.md` (1,083 lines) answering all six questions.
- ✅ **Q1 — Core matching strategy** documented: two-stage `hashret()` hash-bucket pre-filter → `answers()` field confirmation; the returning error is re-dissected into `IPerror`/`ICMPerror` sub-layers.
- ✅ **Q2 — Hashing mechanism** documented byte-exact: request and error produce **byte-identical** `hashret()` values; layout and XOR arithmetic independently verified.
- ✅ **Q3 — TTL/checksum tolerance** demonstrated: neither field participates in the hash or the field comparison, so in-flight mutation does not break matching.
- ✅ **Q4 — Config toggles** demonstrated: `conf.checkIPsrc` and `conf.check_TCPerror_seqack` change accept/reject outcomes; both reverted to defaults.
- ✅ **Q5 — Byte-swapped IP ID** reproduced faithfully: the `socket.htons` transform (`0x1234→0x3412`) gated on `conf.checkIPID`; run-to-run behavior reported honestly.
- ✅ **Q6 — RFC 4884 extensions** demonstrated: `load_contrib('icmp_extensions')` changes the dissected structure but not the match outcome.
- ✅ **Read-only mandate perfectly honored** — zero source files modified; temporary observation scripts cleaned up; working tree clean.
- ✅ **Cross-runtime validated** — reproduced byte-for-byte under host Python 3.13.7 and Docker Python 3.11.13; all 60 `file:line` citations verified exact.

### 1.4 Critical Unresolved Issues

There are no critical unresolved issues. All AAP deliverables are complete, all runtime observations reproduce, and there are no compilation, import, or runtime errors.

| Issue | Impact | Owner | ETA |
|---|---|---|---|
| _None_ | _No blocking issues; deliverable complete and validated across two runtimes_ | — | — |

### 1.5 Access Issues

**No access issues identified.** The repository is fully accessible, the working tree is clean, Python 3.13.7 and Docker are available, and the referenced Scapy checkout is present. The investigation requires no credentials, network egress, or third-party API access (packets are constructed in-process; none are sent on the wire).

| System/Resource | Type of Access | Issue Description | Resolution Status | Owner |
|---|---|---|---|---|
| _None_ | — | _No access issues — all resources available; no credentials required_ | N/A | — |

### 1.6 Recommended Next Steps

1. **[Medium]** Perform an SME technical review and sign-off of the six answers, verifying each against the cited `file:line` anchors in the checkout. *(2.0h)*
2. **[Medium]** Explicitly reconcile the **Q5 determinism finding** — the document reports the byte-swap path as *deterministic* and relocates the reported "sometimes" to the `conf.checkIPID` state plus the `socket.htons` transform — with the original prompt's a-priori framing. *(included in review)*
3. **[Low]** Optionally reproduce the embedded observation scripts in the reviewer's environment, expecting the documented environment-specific first-four-hash-byte variation. *(1.0h)*
4. **[Medium]** Review and merge the pull request (base→HEAD diff is exactly one added file). *(1.0h)*

---

## 2. Project Hours Breakdown

### 2.1 Completed Work Detail

All completed work traces to specific AAP requirements (question IDs Q1–Q6, methodology rules, and scope/hygiene mandates). Hours reflect runtime investigation, byte-level analysis, and evidence-grounded authoring.

| Component | Hours | Description |
|---|---|---|
| Canonical runtime & environment evidence | 2.5 | Establish `PYTHONPATH` import setup; capture interpreter identity, `scapy.__version__`/`__file__`, version-drift note, two byte-identical checkouts (AAP A1–A3) |
| Q1 — Core matching strategy | 2.5 | Faithful `SndRcvHandler` two-stage `hashret()`→`answers()` reproduction; re-dissection to `IPerror`/`ICMPerror`; negative control |
| Q2 — Hashing mechanism | 2.0 | Byte-exact `hashret()` decode; `strxor(src,dst)+proto+ICMP.hashret()` layout; XOR arithmetic; request==error byte-identity |
| Q3 — TTL/checksum tolerance | 1.5 | Four TTL/checksum mutation scenarios; grounding in absence of both fields from hash and comparison |
| Q4 — Configuration toggles | 2.0 | `conf.checkIPsrc` and `conf.check_TCPerror_seqack` before/after; source-IP and seq/ack simulation; revert to defaults |
| Q5 — Byte-swapped IP ID | 3.0 | Faithful 200×, two-process reproduction; value sweep; `socket.htons` transform; `conf.checkIPID` gating; honest determinism finding |
| Q6 — RFC 4884 extensions | 3.0 | `load_contrib` before/after; identical-wire SHA-256 across processes; structure change vs. unchanged match; checksum validation |
| Cross-runtime validation | 1.5 | Docker Python 3.11.13 byte-for-byte cross-check; invariant confirmation across runtimes |
| Coverage pass + citation table | 2.5 | Verify all 60 `file:line` anchors via `grep -n`; assemble anchor table; 8-item named-item coverage matrix |
| Deliverable authoring & synthesis | 3.0 | 1,083-line markdown: summary, how-to-reproduce, per-question structure, cause→effect rationale, observed/inferred discipline |
| Code-review remediation (F1–F7) | 2.0 | Address code-review findings F1–F7 and F1 off-by-one anchor fix (commits `53d519b7`, `f2734cc2`) |
| Final validation | 2.5 | Five production gates: re-run all observations in both runtimes, programmatic 60/60 citation audit, temp-script cleanup, venv-note defect fix, commit (`51225ee6`) |
| **Total Completed** | **28.0** | |

### 2.2 Remaining Work Detail

All remaining work is the standard human path-to-acceptance for a documentation deliverable. There is no remaining AAP-implementation work and there are no blocking (High-priority) tasks.

| Category | Hours | Priority |
|---|---|---|
| Human SME technical review & sign-off of Q1–Q6 answers (accuracy, completeness, evidence sufficiency; reconcile Q5 determinism finding) | 2.0 | Medium |
| Independent reproduction of runtime observations in reviewer environment (optional; expect env-specific hash-byte variation) | 1.0 | Low |
| Pull-request review & merge to target branch | 1.0 | Medium |
| **Total Remaining** | **4.0** | |

### 2.3 Hours Reconciliation

| Check | Result |
|---|---|
| Section 2.1 total (Completed) | 28.0h |
| Section 2.2 total (Remaining) | 4h |
| 2.1 + 2.2 = Total (Section 1.2) | 28 + 4 = **32h** ✅ |
| Completion % = 28 ÷ 32 | **87.5%** ✅ |

---

## 3. Test Results

For this read-only QnA investigation, the "tests" are the **runtime observation scripts** executed through Scapy's real public API. All tests below originate from Blitzy's autonomous validation logs for this project and were re-confirmed during this assessment. There was no traditional unit-test development in scope; the repository's existing `.uts` suites were left untouched (read-only mandate).

| Test Category | Framework | Total Tests | Passed | Failed | Coverage % | Notes |
|---|---|---|---|---|---|---|
| Runtime observation (Q1–Q6) | Python 3.13.7 + Scapy public API | 6 | 6 | 0 | 100% of six questions | Each Q re-dissects/hashes/compares through real `hashret()`/`answers()`/constructors |
| Primary-scenario smoke test | Python 3.13.7 + Scapy | 1 | 1 | 0 | Core path | `hash(orig)==hash(err)==02e40ebd0100000000`; `answers=1` |
| Negative control | Python 3.13.7 + Scapy | 1 | 1 | 0 | Bucket-miss path | Unrelated reply hashes differently and misses the bucket |
| Cross-runtime fidelity | Docker Python 3.11.13 + Scapy | 6 | 6 | 0 | All Q1–Q6 | Byte-for-byte identical invariants; only env-specific first-4 hash bytes differ |
| Citation audit | `grep -n` programmatic | 60 | 60 | 0 | All `file:line` anchors | 60/60 anchors verified exact against `0925ada4` |
| **Total** | | **74** | **74** | **0** | **100%** | Zero failures across all categories |

> **Integrity note:** All results originate from Blitzy's autonomous validation runs and were independently re-verified during this assessment (the primary smoke test and full citation audit were re-executed here).

---

## 4. Runtime Validation & UI Verification

This is a library-internals investigation with **no UI component**; UI verification is not applicable. Runtime health was validated by exercising the real matching code paths.

- ✅ **Operational — Scapy import (host Py 3.13.7):** `import scapy` and `from scapy.all import *` succeed; `scapy.__version__` = `2026.07.10`, resolving to the checkout.
- ✅ **Operational — Scapy import (Docker Py 3.11.13):** imports cleanly in the exact-fidelity container.
- ✅ **Operational — Q1 two-stage match:** error re-dissects to `IP/ICMP/IPerror/ICMPerror`; hash-bucket HIT → `answers()=1`.
- ✅ **Operational — Q2 hash byte-identity:** `orig.hashret() == err.hashret()` (byte-for-byte), decomposed and XOR-verified.
- ✅ **Operational — Q3 TTL/checksum tolerance:** all four mutation scenarios → `answers()=1`.
- ✅ **Operational — Q4 config toggles:** `checkIPsrc` True→0 / False→1; `check_TCPerror_seqack` False→1 / True→0; both reverted.
- ✅ **Operational — Q5 byte-swap:** `{1: 200}` distribution across two processes; transform `0x1234→0x3412` under `checkIPID`.
- ✅ **Operational — Q6 RFC 4884:** layer tree gains `ICMPExtensionHeader`/`ICMPExtensionMPLS` after `load_contrib`; `answers()` unchanged (1→1).
- ⚠ **Partial — Directed Q5 "sometimes":** the anticipated run-to-run non-determinism does **not** reproduce; the path is deterministic. Reported honestly with full evidence (not a defect — correct application of the reproduce-faithfully methodology).
- ✅ **Operational — API integration:** no external services involved; `load_contrib('icmp_extensions')` uses the in-repo contrib module. No outbound network calls.

---

## 5. Compliance & Quality Review

Cross-mapping of AAP deliverables and governing-rule mandates to their verification status. Fixes applied during autonomous validation are noted.

| Benchmark / AAP Mandate | Status | Progress | Evidence |
|---|---|---|---|
| Deliverable at `blitzy/documentation/<branch>.md` | ✅ Pass | 100% | `blitzy/documentation/scapy_0925ada48540.md` exists (1,083 lines) |
| Answer all six questions (Q1–Q6) | ✅ Pass | 100% | Six dedicated sections, each with (a)–(g) structure |
| Answer every named item + coverage pass | ✅ Pass | 100% | 8-item coverage table (core strategy, hashing, TTL, checksum, source-IP, seq/ack, byte-swap, RFC 4884) |
| Run-first: complete unedited output per condition | ✅ Pass | 100% | 10 unedited output blocks with exact commands |
| Byte-exact verification of hashes/checksums | ✅ Pass | 100% | Q2 hash decode; Q6 wire SHA-256 identity across processes |
| Faithful Q5 run-to-run reproduction | ✅ Pass | 100% | Same input 200× across two processes; determinism reported honestly |
| `file:line` citations naming exact function/method | ✅ Pass | 100% | 60/60 anchors verified exact |
| Observed-vs-inferred labeling | ✅ Pass | 100% | Six observed/inferred blocks (one per question) |
| Read-only: zero source modifications | ✅ Pass | 100% | `git diff 0925ada4 HEAD -- scapy/` empty; only doc added |
| No committed code other than the answer document | ✅ Pass | 100% | Base→HEAD = single file `A` |
| Temporary-script hygiene (create under `/tmp`, remove) | ✅ Pass | 100% | Zero lingering scripts; hygiene appendix present |
| Clean working tree on delivery | ✅ Pass | 100% | `git status --porcelain` empty at `51225ee6` |
| **Fix applied:** venv/site-packages mechanism note corrected | ✅ Resolved | 100% | Committed `51225ee6`; conclusion was true, mechanism claim corrected |
| **Fix applied:** code-review findings F1–F7 + anchor off-by-one | ✅ Resolved | 100% | Committed `53d519b7`, `f2734cc2` |

**Outstanding compliance items:** none. All mandates satisfied; two defects found during autonomous validation were fixed and committed.

---

## 6. Risk Assessment

Risks are categorized per the technical / security / operational / integration framework. For a completed read-only documentation deliverable, all identified risks are **Low** severity or **Not Applicable**.

| Risk | Category | Severity | Probability | Mitigation | Status |
|---|---|---|---|---|---|
| Interpreter version drift (observed Py 3.13.7 / 3.11.13; AAP referenced Py 3.12.3, unavailable) | Technical | Low | Low | Version-drift note + Docker cross-runtime check proving interpreter-independence | Mitigated / Documented |
| `scapy.__version__` date-drift (`2026.07.10` vs AAP `2026.07.09`) | Technical | Low | Low | Version is date-sourced; reported exactly as observed | Documented |
| Environment-specific hash bytes (first 4 bytes = `strxor(src,dst)` depend on routed source IP) | Technical | Low | Medium | Flagged "do not hardcode"; invariant (request==error byte-identity) holds regardless of environment | Mitigated / Documented |
| Q5 determinism divergence from AAP's a-priori "sometimes" framing | Technical | Low | Low | Full 200×, two-process evidence + cause→effect; relocated "sometimes" to `checkIPID` + `htons` | Resolved / Documented (SME awareness) |
| No source code changed; no dependencies added; no secrets/credentials | Security | None | N/A | Read-only documentation task; nothing to secure | N/A |
| Deliverable intentionally not wired into the Sphinx build (standalone by AAP design) | Operational | Low | Low | By design; a human may link it if broader discoverability is desired | By-design / Documented |
| Temp-script hygiene must remain clean if reviewers re-run scripts | Operational | Low | Low | Hygiene appendix + development-guide cleanup instructions | Mitigated |
| No external services / APIs / keys / network (packets built in-process; contrib is in-repo) | Integration | None | N/A | Nothing to integrate | N/A |

---

## 7. Visual Project Status

**Project hours breakdown** (Completed = Dark Blue `#5B39F3`, Remaining = White `#FFFFFF`):

```mermaid
pie showData title Project Hours (Total 32h)
    "Completed Work" : 28.0
    "Remaining Work" : 4
```

**Remaining hours by category** (from Section 2.2; totals **4h**):

```mermaid
pie showData title Remaining Work by Category (4h)
    "SME technical review & sign-off" : 2.0
    "Independent reproduction" : 1.0
    "PR review & merge" : 1.0
```

**Remaining work by priority:**

| Priority | Hours | Share of Remaining |
|---|---|---|
| High | 0.0 | 0% |
| Medium | 3.0 | 75.0% |
| Low | 1.0 | 25.0% |
| **Total** | **4.0** | **100%** |

> **Integrity:** "Remaining Work" = **4h**, identical to Section 1.2 (Remaining Hours) and the Section 2.2 "Hours" sum. "Completed Work" = **28h**, identical to Section 2.1. Together = **32h** Total.

---

## 8. Summary & Recommendations

**Achievements.** The project delivers a complete, evidence-grounded answer document (1,083 lines) that explains — with runtime observation as primary evidence — exactly how Scapy matches a returning ICMP destination-unreachable error to the original echo-request. All six questions are answered with the mandated run-first methodology: exact commands, complete unedited output, byte-exact hash verification, `file:line` citations to the precise functions/methods, and clear observed-vs-inferred labeling. The read-only mandate was honored perfectly — zero source files were modified and all temporary observation scripts were cleaned up.

**Remaining gaps.** No AAP-implementation work remains. The **4 hours** of remaining effort are entirely the human path-to-acceptance: SME technical review and sign-off, optional independent reproduction, and PR merge.

**Critical path to production.** SME review (2h) → optional reproduction (1h) → PR merge (1h). None of these are blocking, and there are no High-priority remediation items.

**Success metrics.** All six questions answered ✅ · all 8 named items covered ✅ · 60/60 citations exact ✅ · byte-for-byte reproduction in two runtimes ✅ · zero source modifications ✅ · clean working tree ✅.

**Production-readiness assessment.** The deliverable is **87.5% complete** on an AAP-scoped basis (28h of 32h). The autonomous work is finished and validated; what remains is human acceptance. One point warrants explicit reviewer attention: the **Q5 determinism finding**, where runtime observation diverges from the prompt's a-priori "sometimes" framing and is reported honestly with full supporting evidence. The project is ready for SME review and merge.

| Metric | Value |
|---|---|
| AAP-scoped completion | 87.5% |
| Completed hours | 28 |
| Remaining hours | 4 |
| Total hours | 32 |
| Blocking issues | 0 |
| Source files modified | 0 |

---

## 9. Development Guide

This is a read-only investigation whose deliverable is a static markdown document — there is **no build or deploy step**. This guide therefore explains how to **reproduce and verify** the runtime observations that underpin the answers. Every command below was tested during this assessment.

### 9.1 System Prerequisites

- **OS:** Linux (Ubuntu 25.10 container used here; any POSIX environment works).
- **Python:** CPython 3.x. Observations were captured under **3.13.7** (host) and **3.11.13** (Docker exact-fidelity). The AAP reference was 3.12.3; behavior is interpreter-independent.
- **Scapy:** the in-repo checkout of branch `scapy_0925ada48540` — no installation required; it is imported directly via `PYTHONPATH`.
- **Optional:** Docker, for the exact-fidelity Python 3.11.13 cross-check.

### 9.2 Environment Setup

The Scapy checkout is imported editable from disk — nothing to `pip install`. Point `PYTHONPATH` at the checkout so `import scapy` resolves to it:

```bash
# Import source (a checkout of branch scapy_0925ada48540 @ 0925ada4)
export REPO=/tmp/blitzy/scapy/scapy_0925ada48540_ca08b7
ls -d "$REPO"   # confirm the checkout is present
```

Confirm resolution (run from a **neutral directory** so the intended checkout is selected deterministically):

```bash
cd /tmp
PYTHONPATH="$REPO" python3 -c "import scapy; print(scapy.__version__, scapy.__file__)"
# -> 2026.07.10 /tmp/blitzy/scapy/scapy_0925ada48540_ca08b7/scapy/__init__.py
```

> **Note:** Running `python3 -c` from the delivery repo root prepends the current directory to `sys.path`, so `import scapy` would resolve to the *local* checkout instead. Both are byte-identical checkouts of the same branch, so results are identical either way — but run from a neutral directory when you want deterministic resolution to `$REPO`.

### 9.3 Dependency Installation

None required for the core matching path. Scapy's core imports with zero external dependencies. On `from scapy.all import *`, a benign `CryptographyDeprecationWarning` (TripleDES) is emitted from `scapy/layers/ipsec.py:573,577`; it is unrelated to ICMP matching and can be suppressed with `2>/dev/null`:

```bash
PYTHONPATH="$REPO" python3 -c "from scapy.all import IP, ICMP; print('scapy.all import OK')" 2>/dev/null
# -> scapy.all import OK
```

### 9.4 Reproduce the Primary Scenario (Q1 + Q2)

```bash
cat > /tmp/repro.py <<'PYEOF'
from scapy.all import IP, ICMP, IPerror, ICMPerror
orig = IP(dst="8.8.8.8", id=0x1234)/ICMP()
err  = IP(bytes(IP(src="8.8.8.8", dst=orig.src)/ICMP(type=3, code=1)/bytes(orig)))
print("layers    :", err.summary())
print("hash(orig):", orig.hashret().hex())
print("hash(err) :", err.hashret().hex())
print("identical :", orig.hashret() == err.hashret())
print("answers   :", err.answers(orig))
PYEOF
cd /tmp && PYTHONPATH="$REPO" python3 /tmp/repro.py 2>/dev/null
rm -f /tmp/repro.py   # temp-script hygiene: always clean up
```

Expected output (first 4 hash bytes are environment-specific):

```
layers    : IP / ICMP 8.8.8.8 > <your-src-ip> dest-unreach host-unreachable / IPerror / ICMPerror
hash(orig): 02e40ebd0100000000
hash(err) : 02e40ebd0100000000
identical : True
answers   : 1
```

### 9.5 Verification Steps

- **Import resolves to the checkout:** `scapy.__file__` points under `$REPO` (or a byte-identical checkout).
- **Hash byte-identity (the key invariant):** `orig.hashret() == err.hashret()` prints `True`. The *absolute value* varies by environment (its first 4 bytes are `strxor(src,dst)` of the routed source IP); the *equality* does not.
- **Match confirmation:** `err.answers(orig)` prints `1`.
- **Read-only scope:** from the delivery repo, `git diff --name-status 0925ada4 HEAD` prints exactly `A blitzy/documentation/scapy_0925ada48540.md`, and `git diff --name-status 0925ada4 HEAD -- scapy/` is empty.

### 9.6 View the Deliverable

```bash
DOC=/tmp/blitzy/scapy/blitzy-3f5cfafe-471a-4fd3-a88f-6526e04f5244_eb55fe/blitzy/documentation/scapy_0925ada48540.md
wc -l "$DOC"          # 1083
sed -n '1,40p' "$DOC" # read the summary + how-to-reproduce
```

### 9.7 Optional — Exact-Fidelity Docker Cross-Check (Python 3.11.13)

```bash
docker run --rm --entrypoint bash \
  -v "$REPO":/app -v /tmp/repro.py:/tmp/x.py \
  ghcr.io/scaleapi/swe-atlas:swe_atlas_QnA_secdev_scapy_1.0 \
  -lc 'cd /app && PYTHONPATH=/app python3 /tmp/x.py'
```

### 9.8 Troubleshooting

| Symptom | Cause | Resolution |
|---|---|---|
| `import scapy` finds the wrong copy | `sys.path` order (cwd prepended by `-c`) | Set `PYTHONPATH="$REPO"` and run from a neutral directory (e.g. `cd /tmp`); verify `scapy.__file__` |
| `CryptographyDeprecationWarning` (TripleDES) on stderr | Benign warning from `scapy/layers/ipsec.py:573,577` | Unrelated to ICMP matching; suppress with `2>/dev/null` |
| First 4 hash bytes differ from the doc | `strxor(src,dst)` depends on the routed source IP | Expected and documented; verify the **equality** `orig.hashret()==err.hashret()`, not the absolute bytes |
| `answers()` returns `0` unexpectedly | A `conf` toggle left flipped (e.g. `checkIPsrc`, `checkIPID`) | Restore defaults; the doc's Q4/Q5 sections show the exact toggle states and revert them |
| Leftover scripts under `/tmp` | Observation scripts not cleaned up | Remove them (`rm -f /tmp/<script>.py`); never place scripts inside the repository |

---

## 10. Appendices

### Appendix A — Command Reference

| Command | Purpose |
|---|---|
| `export REPO=/tmp/blitzy/scapy/scapy_0925ada48540_ca08b7` | Set the import-source checkout path |
| `PYTHONPATH="$REPO" python3 -c "import scapy; print(scapy.__version__, scapy.__file__)"` | Confirm Scapy resolves to the checkout |
| `PYTHONPATH="$REPO" python3 <script> 2>/dev/null` | Canonical invocation for observation scripts |
| `git diff --name-status 0925ada4 HEAD` | Verify the base→HEAD change set (single file) |
| `git diff --name-status 0925ada4 HEAD -- scapy/` | Confirm the Scapy source tree is untouched (empty) |
| `wc -l blitzy/documentation/scapy_0925ada48540.md` | Confirm deliverable line count (1083) |
| `git status --porcelain` | Confirm a clean working tree |

### Appendix B — Port Reference

Not applicable. No servers, sockets, or listening ports are used; ICMP packets are constructed in-process and none are transmitted.

### Appendix C — Key File Locations

| Path | Role |
|---|---|
| `blitzy/documentation/scapy_0925ada48540.md` | **The deliverable** (only file added; 1,083 lines) |
| `scapy/sendrecv.py` | Reference — `SndRcvHandler` two-stage match loop (Q1) [L244, L270–L280] |
| `scapy/layers/inet.py` | Reference — `IP`/`ICMP`/`IPerror`/`TCPerror`/`UDPerror`/`ICMPerror` `hashret()`/`answers()` (Q1–Q6) [L568–L611, L986–L1006, L1013–L1095] |
| `scapy/config.py` | Reference — `conf` toggle defaults (Q4, Q5) [L748, L751, L752, L755, L758] |
| `scapy/packet.py` | Reference — base `Packet.hashret`/`Packet.answers` delegation (Q1, Q2) [L1215–L1226] |
| `scapy/contrib/icmp_extensions.py` | Reference — RFC 4884 parsing, `load_contrib` (Q6) [L66–L99, L176–L181] |

### Appendix D — Technology Versions

| Component | Version | Notes |
|---|---|---|
| Scapy (checkout) | `2026.07.10` | Date-sourced; branch `scapy_0925ada48540` @ `0925ada4` |
| CPython (host) | 3.13.7 | Primary observation runtime |
| CPython (Docker) | 3.11.13 | Exact-fidelity cross-check runtime |
| CPython (AAP reference) | 3.12.3 | Not present on host; behavior is interpreter-independent |
| `socket` (stdlib) | bundled | `socket.htons()` for the IP-ID byte-swap comparison |
| `struct` (stdlib) | bundled | `hashret()` byte assembly |

### Appendix E — Environment Variable Reference

| Variable | Value | Purpose |
|---|---|---|
| `PYTHONPATH` | `/tmp/blitzy/scapy/scapy_0925ada48540_ca08b7` | Ensures `import scapy` resolves to the checkout rather than any installed copy |
| `REPO` | (same as above) | Convenience variable used in this guide |

Scapy runtime toggles exercised (read via the global `conf` object, reverted after use):

| `conf` attribute | Default | Used in |
|---|---|---|
| `conf.checkIPsrc` | `True` | Q4 (source-IP validation in the citation) |
| `conf.check_TCPerror_seqack` | `False` | Q4 (TCP seq/ack verification) |
| `conf.checkIPID` | `False` | Q5 (IP-ID / byte-swap comparison) |

### Appendix F — Developer Tools Guide

| Tool | Usage in this project |
|---|---|
| `grep -n` | Programmatic verification of all 60 `file:line` citation anchors against `0925ada4` |
| `git diff` / `git status` | Confirm the read-only mandate (single file added, clean tree) |
| Docker | Exact-fidelity Python 3.11.13 cross-runtime reproduction |
| `python3` (Scapy public API) | Runtime observation via `hashret()`, `answers()`, packet constructors, `load_contrib` |
| `hashlib.sha256` | Q6 wire-identity proof (identical oversized ICMP wire across two processes) |

### Appendix G — Glossary

| Term | Definition |
|---|---|
| `hashret()` | Scapy method returning a hash that is **identical** for a request and its answer; stage one of the match (fast bucketing) |
| `answers()` | Scapy method returning true if one packet is an answer to another; stage two (authoritative field comparison) |
| `IPerror` / `ICMPerror` / `TCPerror` / `UDPerror` | Sub-layer classes into which Scapy auto-decodes the *original datagram* embedded inside an ICMP error |
| Two-stage matching | `hashret()` hash-bucket pre-filter followed by `answers()` field confirmation; a candidate is an answer only if it passes both |
| `conf` | Scapy's global configuration object (`scapy/config.py`) holding the matching toggles |
| `socket.htons` | 16-bit byte swap `((id<<8)|(id>>8)) & 0xFFFF`; the transform behind byte-swapped IP-ID tolerance |
| RFC 4884 | "Extended ICMP to Support Multi-Part Messages"; the extension structure parsed by `load_contrib('icmp_extensions')` when ICMP length exceeds 144 octets |
| Citation | The copy of the original IP header + partial payload embedded inside an ICMP error message |

---

*Completion basis: AAP-scoped hours (PA1). Completed 28h ÷ Total 32h = 87.5%. Completed = Dark Blue `#5B39F3`; Remaining = White `#FFFFFF`.*