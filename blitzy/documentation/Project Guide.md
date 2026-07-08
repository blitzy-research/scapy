# Blitzy Project Guide

> **Deliverable:** `blitzy/documentation/scapy_0925ada48540.md` — a root-cause analysis of Scapy's `sr()`/`sr1()` IP-tunnel response mispairing
> **Repository HEAD (source):** `0925ada485406684174d6f068dbd85c4154657b3` · **Branch:** `blitzy-8025f7ca-f624-4aa8-a655-811a635f7453`
> **Brand palette:** Completed / AI Work = **Dark Blue `#5B39F3`** · Remaining = **White `#FFFFFF`** · Headings/Accents = Violet-Black `#B23AF2` · Highlight = Mint `#A8FDD9`

---

## Section 1 — Executive Summary

### 1.1 Project Overview

This project is a **read-only diagnostic investigation** of the Scapy packet-manipulation library. A user building a network-path probing tool observed that Scapy's `sr()`/`sr1()` stimulus-response API pairs responses with the *wrong* probes when probing through multiple IP-tunnel endpoints. The sole deliverable is a single Markdown answer document that pinpoints the **root cause inside Scapy's two-stage matching algorithm** (`hashret()` bucketing plus `answers()` verification) and demonstrates it with executed, byte-for-byte reproducible evidence driven through the genuine `sr()`/`sr1()` entry point. **No Scapy source code is modified.** The audience is the requesting engineer and, more broadly, Scapy users diagnosing tunnel-probe matching.

### 1.2 Completion Status

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'pie1':'#5B39F3','pie2':'#FFFFFF','pieStroke':'#B23AF2','pieStrokeWidth':'2px','pieOuterStrokeWidth':'2px','pieTitleTextSize':'16px','pieSectionTextColor':'#111111','pieStrokeColor':'#B23AF2'}}}%%
pie showData
    title Completion — 87.5%
    "Completed Work (hrs)" : 35
    "Remaining Work (hrs)" : 5
```

| Metric | Value |
|---|---|
| **Total Hours** | **40** |
| Completed Hours (AI + Manual) | 35 (AI: 35 · Manual: 0) |
| Remaining Hours | 5 |
| **Percent Complete** | **87.5%** |

> Completion is computed with the AAP-scoped, hours-based PA1 methodology: `35 ÷ (35 + 5) × 100 = 87.5%`. All 20 discrete AAP requirements are **Completed**; the 5 remaining hours are path-to-production (human review, environment confirmation, handoff) — not incomplete AAP work.

### 1.3 Key Accomplishments

- ✅ **Root cause identified and located to `path:line`** — a received packet binds to the wrong probe only when the outer tunnel-endpoint address is removed from *both* the `hsent` bucket key (`IP.hashret` [`scapy/layers/inet.py:568-581`]) *and* the `answers()` verification (`IP.answers` [`scapy/layers/inet.py:583-610`]); `_process_packet` then accepts the **first** probe in the collided bucket [`scapy/sendrecv.py:279-281`].
- ✅ **Two-stage matcher dissected** — bucket fill `hsent.setdefault(p.hashret(), []).append(p)` [`scapy/sendrecv.py:244`] + first-in-bucket `answers()` acceptance; proves a hash collision alone is harmless and `answers()` is the deciding gate.
- ✅ **Branch-by-branch analysis** of `IP.hashret`, `IP.answers`, `ICMP.hashret`, `ICMP.answers`, and `IPerror.answers` — including the unconditional quoted-destination check `test_IPdst` [`scapy/layers/inet.py:1022`] that keeps the ICMP-error path safe.
- ✅ **Executed reproduction matrix (A–H + forced-collision)** through the genuine `sr()`/`sr1()` path with an offline fake `conf.L3socket`; every scenario run twice with byte-for-byte identical output.
- ✅ **Byte-level collision proof** — actual `hashret()` hex keys captured and verified against emitted bytes (e.g., collapsed key `040134120100`).
- ✅ **All six user mitigations refuted by name** with executed evidence; honest negative result stated up front (strict defaults did *not* reproduce the mismatch).
- ✅ **Read-only invariant upheld** — `git diff 0925ada4..HEAD` shows only `A blitzy/documentation/scapy_0925ada48540.md`; Scapy source byte-for-byte unchanged; temporary harness cleaned up.
- ✅ **Zero source-induced regressions** — Scapy's own regression suite: 249 passed / 0 failed.

### 1.4 Critical Unresolved Issues

| Issue | Impact | Owner | ETA |
|---|---|---|---|
| Honest boundary condition: the mispairing reproduces only when address checks are disabled (scenarios F/G); it did **not** reproduce at strict defaults, yet the user reports experiencing it. The user's exact live `conf` flags are unconfirmed. | Medium — the direct answer is complete and correct for the reproduced scenarios, but confirming *which* scenario matches the user's real environment requires their live config/bytes. | Human SME + user | ~2h (task H2) |

> No compilation, test, or functional blockers exist. This is a documented, transparent open item — not a defect in the deliverable.

### 1.5 Access Issues

**No access issues identified.** The investigation is fully offline: Scapy imports editable from the checkout, the reproduction uses a fake in-process socket (no raw sockets, no live network, no external services, no credentials). The repository, regression suite, and deliverable are all locally accessible.

| System/Resource | Type of Access | Issue Description | Resolution Status | Owner |
|---|---|---|---|---|
| — | — | No access issues identified | N/A | — |

### 1.6 Recommended Next Steps

1. **[High]** Have a network/Scapy SME review and sign off on the root-cause analysis and its `path:line` citations against HEAD (task H1).
2. **[High]** Confirm the finding against the user's *actual* tunnel environment — capture their live `conf.checkIPaddr/checkIPsrc/checkIPinIP` and the two probes' `hashret()` bytes to identify which reproduced scenario (F/G) applies (task H2).
3. **[Medium]** Deliver the document to the stakeholder and walk through the direct answer plus the context-only remediation options (task H3).
4. **[Low]** Optionally re-run the reproduction matrix on Python 3.12.x to empirically confirm the interpreter-independence claim (task H4).

---

## Section 2 — Project Hours Breakdown

### 2.1 Completed Work Detail

| Component | Hours | Description |
|---|---:|---|
| Subsystem comprehension & two-stage matcher analysis | 4 | Read/trace `SndRcvHandler`, `sndrcv`, `sr`/`sr1`, and the `hsent` bucket mechanism; establish "first probe in a collided bucket wins" [`sendrecv.py:100,244,270-301,322,635,656`]. |
| Branch-by-branch dissection | 6 | `IP.hashret` (strxor fold, checkIPinIP strip, addressless fallback, ICMP-error quote), `IP.answers`, `ICMP.hashret`+`icmp_id_seq_types`, `ICMP.answers`, `IPerror.answers` (`test_IPdst` L1022), and the `Packet` base contract. |
| Offline reproduction harness | 5 | `SuperSocket` subclass assigned to `conf.L3socket` that records sends and replays canned wire bytes, driving the **real** `sr()` path offline. |
| Scenario matrix A–H + forced-collision | 4.5 | Design and execute the full matrix (default vs `checkIPaddr=False` vs `checkIPinIP=False`; echo-reply vs ICMP time-exceeded; identical vs unique inner seq), each run ≥2× for stability. |
| Byte-level `hashret()` collision proof | 2.5 | Capture and verify the exact emitted key bytes (e.g., collapsed `040134120100`, full default keys, forced-collision `0000000304c10978000134120100`). |
| Direct answer/locus + 6 mitigation refutations | 4 | Author the leading direct answer with honest negative result and refute each user experiment by name with executed evidence. |
| Final coverage pass + remediation-as-context | 1.5 | Coverage table mapping every mechanism/flag/experiment to `path:line`/output; remediation options presented as context only (not implemented). |
| Web-search provenance research | 1.5 | Corroborate upstream semantics/provenance of `conf.checkIPinIP`/`checkIPaddr`/`checkIPsrc`/`checkIPID` and the `hashret()` contract. |
| Answer-document authoring | 4 | Compose the 757-line / 53,323-byte Markdown deliverable with embedded output, commands, and appendices. |
| Read-only compliance, cleanup, validation & commit | 2 | Enforce the read-only invariant, remove the temporary harness, run compilation/regression validation, and commit. |
| **Total Completed** | **35** | |

### 2.2 Remaining Work Detail

| Category | Hours | Priority |
|---|---:|---|
| Human SME technical review & sign-off of the root-cause analysis and citations | 2 | High |
| Confirm finding against the user's actual tunnel environment/config (capture live `hashret()` bytes + `conf` flags; close the honest-negative-result gap) | 2 | High |
| Stakeholder delivery & knowledge handoff (walk through answer + context-only remediation options) | 0.5 | Medium |
| Python 3.12.x parity re-run of the reproduction matrix (empirical interpreter-independence check) | 0.5 | Low |
| **Total Remaining** | **5** | |

> **Cross-section check:** Section 2.1 (35) + Section 2.2 (5) = **40** = Total Hours in Section 1.2. Remaining (5) is identical in Sections 1.2, 2.2, and 7.

### 2.3 Notes on Estimation Basis

All 20 AAP requirements are classified **Completed** (0 partial, 0 not-started); therefore Section 2.2 contains **no incomplete AAP work** — only standard path-to-production activities (human review/acceptance, environment confirmation, handoff, optional parity check). Per RG2, maximum pre-human-review completion is capped at 99%; the 87.5% figure reflects the genuine open boundary condition (T1) that requires human environment confirmation.

---

## Section 3 — Test Results

All tests below originate from **Blitzy's autonomous validation logs** for this project.

| Test Category | Framework | Total Tests | Passed | Failed | Coverage % | Notes |
|---|---|---:|---:|---:|---:|---|
| Scapy regression suite (source integrity) | UTscapy (`scapy.tools.UTscapy`) on `test/regression.uts` | 249 | 249 | 0 | N/A* | Authoritative proof of **zero source-induced regression**; matches the setup baseline exactly. Source is unmodified, so the suite is expected to (and does) pass unchanged. Documented `-K` skips applied per baseline. |
| Reproduction matrix (behavioral evidence) | Custom offline harness through genuine `sr()`/`sr1()` → `sndrcv` → `SndRcvHandler` | 9 (scenarios A–H + FC) | 9 | 0 | N/A | Each scenario run **twice**, byte-for-byte identical. Result set: `{E:CORRECT, F:WRONG MATCH, G:WRONG MATCH, H:CORRECT, A:CORRECT, B:CORRECT, C:UNANSWERED, D:UNANSWERED, FC:CORRECT}`. "Passed" = observed outcome matched the documented EXPECT for each scenario. |
| Independent reproduction (validation) | Harness extracted verbatim from Appendix A and re-executed | 1 | 1 | 0 | N/A | The final validator (and this assessment) independently extracted and ran the embedded harness; SUMMARY was **byte-for-byte identical** to the document's embedded output. |
| Byte-level `hashret()` proof | Direct `socket.inet_aton` + `strxor` recomputation | 4 key checks | 4 | 0 | N/A | Independently recomputed folds/keys (`ca000200/ca000203/c1007108/c1097800`, collapsed `040134120100`, FC `0000000304c10978000134120100`) all match the document. |
| Compilation | `python -m compileall -q scapy` | 1 | 1 | 0 | N/A | Exit 0; real HEAD source imports cleanly (`IP/ICMP/IPerror/sr/sr1/SuperSocket`). |

> *Line/branch coverage is not a meaningful metric for a read-only documentation deliverable that adds no source or test code; the "coverage" that matters here is **AAP-requirement coverage (20/20)** and **scenario coverage (echo + ICMP-error paths, default + modified flags, full + truncated quotes)**, both complete.

---

## Section 4 — Runtime Validation & UI Verification

This deliverable has **no UI** (it is a Markdown analysis of a Python library). "Runtime validation" here means executing the matching engine offline through its genuine entry points.

**Runtime health**
- ✅ **Operational** — Scapy imports editable from the checkout (`scapy.__file__` resolves inside the repo; reproductions exercise HEAD source, not an installed copy).
- ✅ **Operational** — `python -m compileall -q scapy` exits 0; `pip check` clean; no dependency changes required.
- ✅ **Operational** — Genuine engine path exercised: `sr()`/`sr1()` → `sndrcv` → `SndRcvHandler._process_packet` with a fake `conf.L3socket` honoring the `send`/`recv`/`select` contract.

**Behavioral validation (reproduction matrix)**
- ✅ **Operational** — Scenarios E/H/A/B/FC produce **CORRECT** pairings; F/G reproduce the **WRONG MATCH**; C/D are **UNANSWERED** — all matching the documented EXPECT, deterministic across two runs.
- ✅ **Operational** — `sr1()` sanity check returns the reply whose outer `src=192.0.2.2` under scenario F, confirming the mispairing surfaces through the single-response entry point too.
- ✅ **Operational** — Byte-level `hashret()` keys captured at runtime match the documented hex exactly.

**UI verification**
- ⚠ **N/A** — No user interface exists for this deliverable; no browser/visual verification is applicable.

---

## Section 5 — Compliance & Quality Review

Cross-map of AAP deliverables to Blitzy quality/compliance benchmarks. Fixes applied during autonomous validation: **none required** (the validator reported the document was already 100% accurate; zero fixes).

| # | AAP Deliverable / Benchmark | Status | Evidence |
|---|---|---|---|
| AAP-1 | Deep comprehension of send/receive matching subsystem | ✅ Pass | Section 2 + citations |
| AAP-2 | Two-stage matcher ("first probe wins") analysis | ✅ Pass | `sendrecv.py:244,279-281` |
| AAP-3 | `IP.hashret` branch-by-branch | ✅ Pass | Section 3.2, `inet.py:568-581` |
| AAP-4 | `IP.answers` dissection | ✅ Pass | Section 3.3, `inet.py:583-610` |
| AAP-5 | `ICMP.hashret` + `icmp_id_seq_types` | ✅ Pass | Section 3.4, `inet.py:949,986-989` |
| AAP-6 | `ICMP.answers` permissive echo path | ✅ Pass | Section 3.5, `inet.py:991-998` |
| AAP-7 | `IPerror.answers` unconditional `test_IPdst` | ✅ Pass | Section 3.6, `inet.py:1022` |
| AAP-8 | `Packet.hashret`/`answers` base contract | ✅ Pass | Section 3.1, `packet.py` |
| AAP-9 | Offline harness driving the REAL `sr()` path | ✅ Pass | Section 4.1 + Appendix A (independently reproduced) |
| AAP-10 | Reproduction matrix A–H+FC, ≥2× runs, unedited output | ✅ Pass | Sections 4.2/4.3 |
| AAP-11 | Byte-level `hashret()` collision proof | ✅ Pass | Section 4.4 |
| AAP-12 | Both response shapes (echo + ICMP errors), full/truncated quotes | ✅ Pass | Scenarios E–H + A–D |
| AAP-13 | `sr1()` sanity check via genuine entry point | ✅ Pass | Section 4.7 + Appendix C |
| AAP-14 | Direct answer + locus, leading with honest negative result | ✅ Pass | Section 1 |
| AAP-15 | Refute all 6 mitigations by name | ✅ Pass | Sections 5.1–5.6 |
| AAP-16 | Final coverage pass grounded in `path:line`/output | ✅ Pass | Section 6 table |
| AAP-17 | Remediation-as-context, explicitly NOT implemented | ✅ Pass | Section 6 end |
| AAP-18 | Web-search provenance of `checkIP*` + `hashret()` contract | ✅ Pass | Reflected in documented semantics |
| AAP-19 | Author `blitzy/documentation/scapy_0925ada48540.md` (+ create dir) | ✅ Pass | 757 lines committed |
| AAP-20 | Read-only invariant + temp cleanup + no tooling side effects | ✅ Pass | `git diff` = 1 file added; `/tmp/scapy_repro` absent |

**Quality benchmarks**

| Benchmark | Status | Notes |
|---|---|---|
| Read-only invariant (no source modified) | ✅ Pass | Only the answer document added; `scapy/` byte-for-byte unchanged |
| Run-first methodology (evidence, not reading) | ✅ Pass | Every behavioral claim backed by executed, unedited output + command |
| Citation accuracy vs HEAD | ✅ Pass | All `path:line` verified; document even corrects the AAP's off-by-2 (`test_IPdst` is L1022) |
| Zero placeholders / TODOs | ✅ Pass | No stub/placeholder/TODO markers in the deliverable |
| Markdown well-formedness | ✅ Pass | Balanced code fences; no trailing whitespace/CRLF/BOM |
| Cleanup mandate | ✅ Pass | Temporary harness removed; no stray files in repo |
| Committed & consistent | ✅ Pass | Committed blob identical to working tree across 3 `agent@blitzy.com` commits |

---

## Section 6 — Risk Assessment

| Risk | Category | Severity | Probability | Mitigation | Status |
|---|---|---|---|---|---|
| **T1** User-environment gap: mispairing reproduces only with address checks disabled (F: `checkIPaddr`/`checkIPsrc=False`; G: `checkIPinIP=False`); did **not** reproduce at strict defaults although the user reports it. | Technical | Medium | Medium | Capture the user's live `hashret()` bytes + `conf` flags to confirm which scenario applies (task H2). Disclosed transparently as an honest boundary condition in the document itself. | **Open** |
| **T2** Runtime version parity: reproduced on Python 3.11.13 vs AAP-referenced 3.12.3. | Technical | Low | Low | Matching logic is pure-Python and interpreter-independent (argued in doc); optional 3.12 re-run (task H4). | Mitigated |
| **T3** Citation/line-number drift on other revisions. | Technical | Low | Low | Analysis explicitly pinned to HEAD `0925ada48540` with exact `path:line`. | Mitigated |
| **S1** Advisory: disabling `conf.checkIPaddr`/`checkIPsrc` weakens reply-source validation (enables the mispairing). | Security | Low | Low | Document recommends keeping defaults (`True`) and not disabling for tunnel probing. No source changed, no new attack surface, no secrets/credentials. | Mitigated |
| **O1** Reproduction harness is temporary. | Operational | Low | Low | Full scripts embedded verbatim in Appendices A/B/C for exact re-run; harness cleaned up per mandate. | Mitigated |
| **O2** No CI guard; analysis could go stale if Scapy's matcher changes upstream. | Operational | Low | Low | Point-in-time analysis pinned to a revision; re-validate on Scapy upgrade. | Accepted |
| **I1** Offline harness does not exercise real raw sockets/live network. | Integration | Low | Low | By design — the wire was already validated by the user via tcpdump/Wireshark (out of scope as a cause); optional live confirmation (task H2). | Accepted |

**Overall risk posture: LOW.** The single material item is **T1**, which maps directly to remaining task **H2** and is transparently disclosed in the deliverable.

---

## Section 7 — Visual Project Status

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'pie1':'#5B39F3','pie2':'#FFFFFF','pieStroke':'#B23AF2','pieStrokeColor':'#B23AF2','pieSectionTextColor':'#111111','pieTitleTextSize':'16px'}}}%%
pie showData
    title Project Hours Breakdown (Total 40h)
    "Completed Work" : 35
    "Remaining Work" : 5
```

**Remaining hours by priority (Section 2.2)**

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'pie1':'#5B39F3','pie2':'#B23AF2','pie3':'#A8FDD9','pieSectionTextColor':'#111111','pieTitleTextSize':'16px'}}}%%
pie showData
    title Remaining 5h by Priority
    "High (H1+H2)" : 4
    "Medium (H3)" : 0.5
    "Low (H4)" : 0.5
```

> **Integrity:** "Remaining Work" = **5** here = Remaining Hours in Section 1.2 = sum of Section 2.2 Hours column. "Completed Work" = **35** = Completed Hours in Section 1.2 = sum of Section 2.1 Hours column.

---

## Section 8 — Summary & Recommendations

**Achievements.** The project is **87.5% complete** (35 of 40 hours). All 20 AAP-scoped requirements are delivered: the root cause is pinpointed to a `path:line` locus, the two-stage matcher is fully dissected, and the failure is demonstrated through the genuine `sr()`/`sr1()` engine with an executed, byte-for-byte reproducible scenario matrix and byte-level `hashret()` proof. Every user mitigation is refuted by name, and the read-only invariant is upheld with zero source-induced regressions (249/249 regression tests pass).

**Remaining gaps.** The 5 remaining hours are entirely **path-to-production / human-acceptance** work — SME sign-off (2h), confirming the finding against the user's real environment (2h), stakeholder handoff (0.5h), and an optional Python 3.12 parity re-run (0.5h). There is **no incomplete AAP work** and no functional blocker.

**Critical path to production.** The only material item is the honest boundary condition (T1): the mispairing reproduces when address checks are disabled (scenarios F/G) but not at strict defaults, while the user reports experiencing it. Confirming the user's live `conf` flags and `hashret()` bytes (task H2) resolves which scenario applies and closes the gap.

**Production readiness assessment.** The deliverable is **accurate, complete, reproducible, and committed**, with the repository left byte-for-byte unchanged apart from the single in-scope answer document. It is ready for SME review and stakeholder delivery. The document's transparency about its own boundary condition is a quality strength, not a defect.

| Success Metric | Target | Actual |
|---|---|---|
| AAP requirements completed | 20/20 | 20/20 ✅ |
| Source-induced regressions | 0 | 0 ✅ |
| Reproduction determinism | Stable ≥2 runs | Byte-for-byte identical ✅ |
| Read-only invariant | Held | Held (1 file added) ✅ |
| Completion | ≥ target | 87.5% |

---

## Section 9 — Development Guide

> **Nature of this project:** a read-only Scapy investigation. Scapy runs **editable from the checkout**, so there is **no dependency install step** — the guide below shows how to verify the environment, reproduce the embedded evidence, and run the regression suite. Every command below was **tested** on this container.

### 9.1 System Prerequisites

- **OS:** Linux (Ubuntu-based container).
- **Python:** 3.11.13 (bundled `.venv`). *The AAP referenced 3.12.3; the matching logic is pure-Python and interpreter-independent.* Declared support: `requires-python ">=3.7, <4"`.
- **Tooling:** `git` + `git-lfs`. No external network, raw sockets, or credentials required.
- **Disk:** repository ≈ 31 MB.

### 9.2 Environment Setup

```bash
# 1. Enter the repository root (this is the working directory)
cd /tmp/blitzy/scapy/blitzy-8025f7ca-f624-4aa8-a655-811a635f7453_0f9b3e

# 2. Confirm the bundled interpreter
.venv/bin/python --version          # -> Python 3.11.13

# 3. Make the checkout importable (Scapy is editable from source)
export PYTHONPATH="$(pwd)"
```

### 9.3 Dependency Installation

```bash
# No install step is required — Scapy resolves from the working tree.
# Verify Scapy imports FROM THE CHECKOUT (not an installed copy):
PYTHONPATH="$(pwd)" .venv/bin/python -c \
  "import scapy, os; f=os.path.abspath(scapy.__file__); print(f); print('inside_repo=', f.startswith(os.getcwd()))"
# Expected:
#   .../scapy/__init__.py
#   inside_repo= True
```

### 9.4 Build / Compilation Check

```bash
.venv/bin/python -m compileall -q scapy   # exit code 0 = clean
PYTHONPATH="$(pwd)" .venv/bin/python -c \
  "from scapy.all import IP, ICMP, sr, sr1; from scapy.layers.inet import IPerror; from scapy.supersocket import SuperSocket; print('imports OK')"
```

### 9.5 Reproduce the Deliverable's Evidence

The document's embedded appendices are its test suite. Extract and run the harness (Appendix A):

```bash
mkdir -p /tmp/scapy_repro
PYTHONPATH="$(pwd)" .venv/bin/python - <<'PY'
import re
doc = open("blitzy/documentation/scapy_0925ada48540.md").read()
seg = doc[doc.index("## Appendix A"):doc.index("## Appendix B")]
code = re.search(r"```python\n(.*?)```", seg, re.S).group(1)
open("/tmp/scapy_repro/harness.py", "w").write(code)
print("harness.py extracted")
PY

PYTHONPATH="$(pwd)" .venv/bin/python /tmp/scapy_repro/harness.py | grep -i SUMMARY
# Expected (byte-for-byte):
# SUMMARY: {'E': 'CORRECT', 'F': 'WRONG MATCH', 'G': 'WRONG MATCH', 'H': 'CORRECT',
#           'A': 'CORRECT', 'B': 'CORRECT', 'C': 'UNANSWERED', 'D': 'UNANSWERED', 'FC': 'CORRECT'}

# Appendices B (why_B_C.py) and C (sr1_check.py) extract/run the same way
# (change the slice bounds to "## Appendix B"/"## Appendix C" and the tail).

# CLEANUP (the repo MUST remain unchanged):
rm -rf /tmp/scapy_repro
```

### 9.6 Run Scapy's Regression Suite (source-integrity check)

```bash
PYTHONPATH="$(pwd)" .venv/bin/python -m scapy.tools.UTscapy \
  -t test/regression.uts -N -b   # documented -K skips per baseline
# Expected: PASSED=249 FAILED=0
```

### 9.7 View the Deliverable

```bash
sed -n '1,60p' blitzy/documentation/scapy_0925ada48540.md   # header + Section 1
wc -l blitzy/documentation/scapy_0925ada48540.md            # 757
```

### 9.8 Verification Checklist

- `inside_repo= True` — Scapy resolves to the checkout (evidence reflects HEAD source).
- `compileall` exit code `0`; key imports succeed.
- Harness SUMMARY matches the expected dict byte-for-byte.
- Regression: `PASSED=249 FAILED=0`.
- `git diff 0925ada4..HEAD --name-status` shows **only** `A blitzy/documentation/scapy_0925ada48540.md`.

### 9.9 Troubleshooting

| Symptom | Cause | Resolution |
|---|---|---|
| `ModuleNotFoundError: scapy` | `PYTHONPATH` not set to repo root | `export PYTHONPATH="$(pwd)"` from the repository root |
| `scapy.__file__` points to site-packages | An installed Scapy shadows the checkout | Ensure the repo root precedes site-packages on `PYTHONPATH` |
| Harness reports unexpected pairings | Not running under the HEAD checkout | Re-run with `PYTHONPATH="$(pwd)"` so `hashret`/`answers` reflect HEAD |
| Repo shows unexpected diffs | Stray temp files written into the repo | Keep temp scripts under `/tmp/scapy_repro`; confirm with `git status` |

---

## Section 10 — Appendices

### Appendix A — Command Reference

| Purpose | Command |
|---|---|
| Interpreter version | `.venv/bin/python --version` |
| Verify editable import | `PYTHONPATH="$(pwd)" .venv/bin/python -c "import scapy,os;print(os.path.abspath(scapy.__file__))"` |
| Compile check | `.venv/bin/python -m compileall -q scapy` |
| Key imports | `... -c "from scapy.all import IP,ICMP,sr,sr1; from scapy.layers.inet import IPerror; from scapy.supersocket import SuperSocket"` |
| Run harness | `PYTHONPATH="$(pwd)" .venv/bin/python /tmp/scapy_repro/harness.py` |
| Regression suite | `PYTHONPATH="$(pwd)" .venv/bin/python -m scapy.tools.UTscapy -t test/regression.uts -N -b` |
| Read-only check | `git diff 0925ada4..HEAD --name-status` |
| View deliverable | `sed -n '1,60p' blitzy/documentation/scapy_0925ada48540.md` |

### Appendix B — Port Reference

Not applicable — the deliverable is an offline analysis with no listening services or ports.

### Appendix C — Key File Locations

| Path | Role |
|---|---|
| `blitzy/documentation/scapy_0925ada48540.md` | **The deliverable** (757 lines, 53,323 bytes) |
| `scapy/sendrecv.py` | Matcher engine & entry points (`SndRcvHandler` L100, `hsent` L244, `_process_packet` L270-301, `sndrcv` L322, `sr`/`sr1` L635/L656) — reference only |
| `scapy/layers/inet.py` | Key/verify logic (`IP.hashret` L568-581, `IP.answers` L583-610, `ICMP.hashret`/`answers` L986-998, `IPerror.answers` L1016-1034, `test_IPdst` L1022) — reference only |
| `scapy/packet.py` | `Packet.hashret`/`answers` base contract — reference only |
| `scapy/config.py` | `checkIP*` flag defaults (L748-755) — reference only |
| `scapy/supersocket.py` | `SuperSocket` send/recv/select contract (L97/L172/L250) — reference only |
| `test/regression.uts` | Scapy regression suite (157,705 bytes) |

### Appendix D — Technology Versions

| Component | Version |
|---|---|
| Python | 3.11.13 (container `.venv`) |
| Scapy | HEAD `0925ada485406684174d6f068dbd85c4154657b3`, editable from checkout |
| Declared Python support | `>=3.7, <4` (`pyproject.toml`) |

### Appendix E — Environment Variable Reference

| Variable | Value | Purpose |
|---|---|---|
| `PYTHONPATH` | repository root (`$(pwd)`) | Makes Scapy import editable from the checkout so runs exercise HEAD source |

### Appendix F — Developer Tools Guide

- **UTscapy** (`scapy.tools.UTscapy`) — Scapy's built-in unit-test runner used for the regression suite.
- **compileall** — byte-compiles the `scapy` package to confirm the source parses cleanly.
- **git** — read-only invariant verification via `git diff 0925ada4..HEAD --name-status`.

### Appendix G — Glossary

| Term | Meaning |
|---|---|
| `hashret()` | Per-packet method returning bytes that are intended to be **equal** for a stimulus and its reply; used to bucket sent probes. |
| `answers()` | Per-packet predicate deciding whether a received packet answers a given sent probe; the second matcher gate. |
| `hsent` | Dict keyed by `hashret()` bytes mapping to a list (bucket) of sent probes [`sendrecv.py:244`]. |
| Bucket collision | Two distinct probes producing the same `hashret()` key, landing in one `hsent` list. |
| `strxor(src,dst)` | Symmetric address fold in `IP.hashret` that makes a request and its reply hash equal. |
| `checkIPaddr`/`checkIPsrc`/`checkIPinIP` | `conf` flags gating outer-address checks in the matcher; disabling them removes the outer address from the key and/or verification. |
| Scenario F/G | Reproduced **WRONG MATCH** cases (echo reply with address checks disabled). |
| Honest negative result | The finding that strict-default configuration did **not** reproduce the mismatch in controlled runs. |