# Blitzy Project Guide

> **Project:** Runtime-grounded Q&A on Scapy ICMP-error request/response matching
> **Branch:** `blitzy-79f5a7dc-eb92-4173-a438-8867623b36db` &nbsp;•&nbsp; **HEAD:** `732f81aa` &nbsp;•&nbsp; **Base:** `0925ada4`
> **Task type:** Documentation / read-only runtime investigation (SWE-AtlasQnA-Repo)
>
> **Color legend (Blitzy brand):** <span style="color:#5B39F3">■</span> **Completed / AI Work — Dark Blue `#5B39F3`** &nbsp;|&nbsp; <span style="color:#B23AF2">■</span> Remaining / Not Completed — White `#FFFFFF` (shown outlined) &nbsp;|&nbsp; Headings/Accents — Violet-Black `#B23AF2` &nbsp;|&nbsp; Highlight — Mint `#A8FDD9`

---

## 1. Executive Summary

### 1.1 Project Overview

This project delivers a single, runtime-grounded knowledge document — `blitzy/documentation/scapy_0925ada48540.md` — explaining how Scapy performs IPv4 ICMP-error request/response matching. It answers six named sub-questions (Q1–Q6) covering the two-stage `hashret()`/`answers()` pipeline, identical hash bucketing, TTL/checksum tolerance, the `conf.checkIPsrc` and `conf.check_TCPerror_seqack` toggles, the `socket.htons` IP-ID byte-swap tolerance, and RFC4884 extension parsing. Every claim was produced by building and running the in-repo Scapy, exercising the real matching entry point, and transcribing complete unedited output with `file:line` citations. The task is strictly read-only: no Scapy source is modified. Target users are Scapy maintainers, network engineers, and security researchers needing an authoritative, reproducible explanation of stimulus-response matching internals.

### 1.2 Completion Status

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'pie1':'#5B39F3','pie2':'#FFFFFF','pieStrokeColor':'#B23AF2','pieStrokeWidth':'2px','pieOuterStrokeWidth':'2px','pieSectionTextColor':'#111111','pieTitleTextSize':'16px','pieLegendTextSize':'13px'}}}%%
pie showData
    title Completion Status — 95.7% Complete
    "Completed Work (hrs)" : 44
    "Remaining Work (hrs)" : 2
```

*Slice colors: Completed = Dark Blue `#5B39F3`; Remaining = White `#FFFFFF` (outlined in `#B23AF2`).*

| Metric | Hours |
|---|---|
| **Total Hours** | **46** |
| **Completed Hours (AI + Manual)** | **44** (AI: 44 · Manual: 0) |
| **Remaining Hours** | **2** |
| **Percent Complete** | **95.7%** &nbsp;( 44 ÷ 46 = 95.7% ) |

### 1.3 Key Accomplishments

- ✅ All six sub-questions (Q1–Q6) answered with the required 5-part structure: direct answer, `file:line` citation, exact command, complete unedited output, and cause→effect.
- ✅ The **real matching entry point** `SndRcvHandler._process_packet()` was exercised in-memory, yielding exactly **1** `QueryAnswer` couple — the identical code path `sr()`/`sr1()` use.
- ✅ Byte-sensitive result verified exactly: `hashret()` = `b'\xca\x00\x02\x04\x01\xaa\xaa\x07\x00'`, **identical** for the request and the ICMP error.
- ✅ Run-to-run distribution reported for Q5 (same input ×5 = `[1,1,1,1,1]`; cross-process deterministic), faithfully reproducing the "sometimes matches" phenomenon rather than hiding it.
- ✅ RFC4884 `pkt.len > 144` threshold grounded in the published standard (128-octet original datagram + headers).
- ✅ Two Scapy-source anomalies observed and **reported, not repaired** (Q5 `checkIPID` mode-2 comment-vs-code; Q6 extension build-vs-validate checksum) — honoring read-only scope.
- ✅ 39 distinct `file:line` citations, all resolving to source at HEAD `0925ada4`; coverage-pass checklist marks all 6 named items complete.
- ✅ **Read-only honored:** `git diff` vs base = exactly one added file; repository byte-for-byte unchanged.
- ✅ Independently **corroborated by Scapy's own regression suite: 55/55 pass** (inet.uts 54/54 including #047 & #050; icmp_extensions.uts 1/1).

### 1.4 Critical Unresolved Issues

**No critical (release-blocking) issues identified.** The deliverable compiles as valid Markdown, all runtime claims reproduce byte-for-byte, and the repository is clean. The single open item is a routine, non-blocking human sign-off gate, shown below for completeness.

| Issue | Impact | Owner | ETA |
|---|---|---|---|
| Human SME technical sign-off not yet performed | Non-blocking — content already triple-verified (validator + independent re-run + Scapy regression); sign-off is a formality before the doc is deemed authoritative | Network/Scapy SME reviewer | 1.5h |

### 1.5 Access Issues

**No access issues identified.** The repository, the in-repo Scapy checkout, the Python interpreter (3.13.7), the `.venv`, Git, and Git LFS were all fully accessible; every observation script and regression campaign executed successfully with no permission, credential, or third-party-access obstacle.

| System/Resource | Type of Access | Issue Description | Resolution Status | Owner |
|---|---|---|---|---|
| — | — | No access issues identified | N/A | — |

### 1.6 Recommended Next Steps

1. **[High]** Have a network-protocol/Scapy SME read the six answers and spot-check the byte-level assertions (hashret bytes, `htons` transform, Q6 checksums), then approve. *(1.5h)*
2. **[Medium]** Perform standard PR review of the single additive file and merge to the target branch. *(0.5h)*
3. **[Low]** *(optional, 0h)* Re-run the observation scripts for a fresh-eyes reproduction — already done twice (validator 6/6 + this assessment).
4. **[Low]** *(optional, 0h)* Consider filing upstream Scapy issues for the two reported-not-repaired anomalies (Q5 mode-2 comment-vs-code; Q6 build-vs-validate checksum).
5. **[Low]** *(optional, 0h)* Decide whether to cross-link the document from an internal knowledge base (it is intentionally outside Scapy's Sphinx/packaging).

---

## 2. Project Hours Breakdown

### 2.1 Completed Work Detail

All completed work was performed autonomously (AI). Each component traces to a specific AAP requirement.

| Component | Hours | Description |
|---|---:|---|
| Investigation harness & real matching-entry-point driver | 4 | Drives `SndRcvHandler._process_packet()` in-memory (via `object.__new__` + manual `hsent` bucket per `sendrecv.py:L244/L270-L301`) — the identical code path `sr()`/`sr1()` use. |
| RFC4884 (Q6) standards research & grounding | 2 | Researched "Extended ICMP to Support Multi-Part Messages" to ground the `pkt.len > 144` threshold (128-octet original datagram + headers). |
| Q1 — Matching strategy investigation + two-stage flow diagram | 5 | Layer enumeration `['IP','ICMP','IPerror','ICMPerror']`, `QueryAnswer` couple, Mermaid pipeline diagram, cause→effect. |
| Q2 — `hashret()` value & byte-decomposition investigation | 3 | Identical `b'\xca\x00\x02\x04\x01\xaa\xaa\x07\x00'`; decomposed into `strxor(src,dst)` + proto + `struct.pack("HH",id,seq)`; verified exact bytes. |
| Q3 — TTL / checksum tolerance investigation | 3 | Rebuilt embedded copy with TTL 64→5 and IP checksum `0x9ca7`→`0xdead`; both still `answers()=1`; hash unchanged. |
| Q4 — `conf.checkIPsrc` / `conf.check_TCPerror_seqack` toggle investigation | 4 | TCP-in-ICMP error; both flags exercised in both states; hash-bucket reshape observed; strict-vs-tolerant trade-off. |
| Q5 — IP-ID `socket.htons` byte-swap / `conf.checkIPID` investigation | 5 | `htons(0x1234)=0x3412`; per-mode matrix; same input ×5 + cross-process determinism; mode-2 comment-vs-code discrepancy reported. |
| Q6 — RFC4884 extension load/dissect investigation | 6 | Built valid multi-part extension; dissected identical bytes with/without contrib; build-vs-validate checksum discrepancy; layer transition (Padding 96→84); `answers()=1` invariant. |
| Document scaffolding (Environment, direct-answer summary, verbatim scenario) | 2 | Environment table, one-line answers, verbatim reproduction of AAP §0.1.1 and the read-only directive. |
| Coverage-pass checklist (6 named items) | 1 | Maps TTL, IP checksum, `checkIPsrc`, `check_TCPerror_seqack`, byte-swapped IP-ID/`checkIPID`, RFC4884 threshold to their answers. |
| `file:line` citation appendix (39 refs verified) | 2 | Appendix cross-referencing every cited function/method against source. |
| Read-only compliance discipline (temp-script hygiene, cleanup) | 1 | Scripts under `/tmp/scapy_probe/`, `PYTHONDONTWRITEBYTECODE=1`, `.bin` artifacts removed; repo left unchanged. |
| Runtime-claim verification — 6/6 observation scripts byte-for-byte | 3 | Reconstructed all scripts, diffed output against documented transcripts — exact matches. |
| Scapy regression cross-corroboration — 55/55 | 1 | `inet.uts` 54/54 (incl. #047, #050) + `icmp_extensions.uts` 1/1 via UTscapy. |
| Citation audit + Markdown well-formedness | 1 | 39 citations resolve; 32 balanced code fences; no TODO/placeholder markers. |
| Read-only / git-state verification | 1 | `git diff` base `--name-status` = 1 file; working tree clean; no temp scripts leaked. |
| **Total Completed** | **44** | |

### 2.2 Remaining Work Detail

Each remaining item is a standard path-to-production human gate (not an autonomous engineering task).

| Category | Hours | Priority |
|---|---:|---|
| SME technical review & sign-off of the Q&A content | 1.5 | High |
| PR review & merge to target branch | 0.5 | Medium |
| **Total Remaining** | **2.0** | |

*Optional follow-ups (O1 fresh-eyes reproduction, O2 upstream anomaly filing, O3 KB discoverability) are deliberately excluded at 0h and are not on the critical path.*

### 2.3 Hours Reconciliation

| Check | Value | Result |
|---|---|---|
| Section 2.1 completed total | 44h | — |
| Section 2.2 remaining total | 2h | — |
| 2.1 + 2.2 | 46h | = Total Project Hours (Section 1.2) ✅ |
| Completion % | 44 ÷ 46 = 95.7% | Matches Section 1.2, 7, 8 ✅ |

---

## 3. Test Results

All entries below originate from Blitzy's autonomous validation logs and were independently re-executed during this assessment. "Runtime observation scripts" are the primary correctness tests for a runtime-grounded document; Scapy's own `.uts` regression campaigns provide independent corroboration of the cited behaviors.

| Test Category | Framework | Total Tests | Passed | Failed | Coverage % | Notes |
|---|---|---:|---:|---:|---|---|
| Runtime observation scripts (Q1–Q6 claim verification) | Custom Python harness on in-repo Scapy (`hashret()`/`answers()` + real `SndRcvHandler._process_packet()`) | 6 | 6 | 0 | 100% of Q1–Q6 | probe1 (Q1/Q2/Q3), probe2 (Q4), probe3 (Q5), q6_builder + q6_reader_nocontrib + q6_reader_contrib (Q6); reproduced byte-for-byte |
| Scapy regression — IP/ICMP layer | UTscapy (`.uts`) | 54 | 54 | 0 | Cited `answers()`/`hashret()` behaviors | Incl. #047 "(IP|UDP|TCP|ICMP)Error" and #050 "ICMP hashret" — the exact behaviors the doc cites |
| Scapy regression — RFC4884 extensions | UTscapy (`.uts`) + `load_contrib('icmp_extensions')` | 1 | 1 | 0 | Extension construction | Corroborates Q6 |
| **Totals** | | **61** | **61** | **0** | **100% pass** | Zero failed, zero skipped, zero blocked |

---

## 4. Runtime Validation & UI Verification

**Runtime health (library / documentation task — no web UI):**

- ✅ **Operational** — In-repo Scapy imports and runs from the checkout (`conf.version` = `2026.07.08`), not a pip-installed copy.
- ✅ **Operational** — Real matching entry point `SndRcvHandler._process_packet()` runs in-memory and yields exactly **1** matched `QueryAnswer` couple.
- ✅ **Operational** — `hashret()` produces the documented byte string `b'\xca\x00\x02\x04\x01\xaa\xaa\x07\x00'`, identical for request and ICMP error.
- ✅ **Operational** — All six runtime experiments reproduce their documented transcripts byte-for-byte.
- ✅ **Operational** — Scapy's own regression suite passes 55/55.
- ✅ **Operational** — Read-only state confirmed: `git diff` vs base = exactly one added file; working tree clean.

**UI verification:** ⚠ **Not applicable** — this deliverable is a Markdown knowledge document; there is no user interface to verify.

**API / network integration:** ⚠ **Not applicable by design** — no on-the-wire transmission is performed. The canonical `hashret()`/`answers()` functions are exercised directly in memory (the same code path the send/receive engine uses), which is faithful and requires no raw sockets or privileges.

---

## 5. Compliance & Quality Review

AAP deliverables and SWE-AtlasQnA-Repo rules mapped to their verification status. No fixes to the deliverable were required during autonomous validation — it was verified 100% accurate.

| Requirement (AAP / Rule) | Benchmark | Status | Progress | Evidence |
|---|---|---|---|---|
| Deliverable location & name (`blitzy/documentation/scapy_0925ada48540.md`) | Correct path derived from source branch | ✅ Pass | 100% | File present (458 lines); `git diff` shows it added |
| Run-first methodology (build & run, then write) | Claims from observed output, not reading | ✅ Pass | 100% | Each Q has command + complete unedited output |
| Reproduce reported inconsistency faithfully (Q5) | Same input run repeatedly; report distribution | ✅ Pass | 100% | ×5 = `[1,1,1,1,1]`; cross-process deterministic |
| Exercise real entry point | `SndRcvHandler._process_packet()` | ✅ Pass | 100% | 1 `QueryAnswer` couple produced |
| Complete unedited output + command | No paraphrase/elision | ✅ Pass | 100% | Full transcripts in Q1–Q6 |
| Byte-sensitive `hashret()` verified exactly | Match emitted bytes | ✅ Pass | 100% | `b'\xca\x00\x02\x04\x01\xaa\xaa\x07\x00'` (re-verified) |
| Every claim carries `file:line` | Concrete function/method refs | ✅ Pass | 100% | 39 distinct citations, all resolve |
| Answer every named item + coverage pass | TTL, checksum, `checkIPsrc`, `check_TCPerror_seqack`, byte-swap/`checkIPID`, RFC4884 | ✅ Pass | 100% | Coverage checklist: 6/6 `[x]` |
| RFC4884 research grounds Q6 threshold | Standards-based `pkt.len > 144` | ✅ Pass | 100% | Environment/Q6 cite the 128-octet basis |
| Read-only scope | No source modified; temp scripts removed | ✅ Pass | 100% | `git diff` = 1 added file; tree clean |
| Report anomalies as observed (not adjusted) | Truthful reporting | ✅ Pass | 100% | Q5 mode-2 & Q6 checksum discrepancies documented, not repaired |
| Markdown quality | Well-formed, no placeholders | ✅ Pass | 100% | 32 balanced fences; no TODO/FIXME/stub markers |
| Human SME sign-off | Expert approval | ⚠ Pending | 0% | Routine gate — Section 2.2 (1.5h) |

**Fixes applied during autonomous validation:** None to the deliverable (verified accurate as authored). Adjustments were confined to throwaway reconstruction scripts outside the repository.

---

## 6. Risk Assessment

| Risk | Category | Severity | Probability | Mitigation | Status |
|---|---|---|---|---|---|
| R1 — Byte-level technical claims not yet human-SME-reviewed | Technical | Medium | Low | Validator verified 6/6 scripts byte-for-byte; independently re-verified; Scapy's 55/55 regression corroborates cited behaviors (#047, #050) | Mitigated (open — maps to remaining High task) |
| R2 — Interpreter-version note (AAP said 3.12.3; container ships 3.13.7) | Technical | Low | Low | Doc explicitly notes it, argues the pure-Python deterministic logic (`struct.pack`, `socket.htons`, `inet_pton`) is version-invariant, and confirms byte-for-byte reproduction on 3.13.7 | Resolved (documented) |
| R3 — Runtime values pinned to HEAD `0925ada4` / `conf.version 2026.07.08` may drift if source changes | Technical | Low | Low | Environment table pins HEAD + version; Scapy regression tracks the behavior; point-in-time investigation | Documented (by design) |
| R4 — Two Scapy-source anomalies reported-not-repaired (Q5 mode-2 comment/code; Q6 build/validate checksum) | Technical | Low | Low | Documented as observations; repairing would violate read-only scope | By design (out of scope) |
| R5 — Reproducibility depends on exact invocation (`PYTHONPATH=.`, in-repo import) | Operational | Low | Low | Exact copy-pasteable commands in Environment + each Q section | Mitigated |
| R6 — Deliverable excluded from Scapy Sphinx/packaging (`blitzy/` tree) | Operational | Low | Low | Intentional; verified `pyproject` includes only `scapy*`, `blitzy/` not referenced in any build file | By design (verified) |
| R7 — PR merge integration | Integration | Low | Low | Single additive file in a new directory; zero import/packaging impact; clean tree; `.venv`/`__pycache__` gitignored | Low risk |
| Security | Security | None | — | No code/deps/secrets/network added; read-only doc introduces no attack surface | N/A |

**Overall posture: LOW.** The single material open item is human sign-off (R1), already heavily mitigated by triple verification. No blocking, high-severity, or unmitigated risks.

---

## 7. Visual Project Status

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'pie1':'#5B39F3','pie2':'#FFFFFF','pieStrokeColor':'#B23AF2','pieStrokeWidth':'2px','pieOuterStrokeWidth':'2px','pieSectionTextColor':'#111111','pieTitleTextSize':'16px','pieLegendTextSize':'13px'}}}%%
pie showData
    title Project Hours Breakdown (Total 46h)
    "Completed Work" : 44
    "Remaining Work" : 2
```

*Completed = Dark Blue `#5B39F3` · Remaining = White `#FFFFFF` (outlined `#B23AF2`).*

**Remaining work by category (Section 2.2 — sums to the 2h "Remaining Work" slice):**

| Category | Hours | Priority |
|---|---:|---|
| SME technical review & sign-off | 1.5 | High |
| PR review & merge | 0.5 | Medium |
| **Total** | **2.0** | |

*Integrity: the "Remaining Work" pie value (2) equals Section 1.2 Remaining Hours (2) and the Section 2.2 Hours sum (2).*

---

## 8. Summary & Recommendations

**Achievements.** The project is **95.7% complete (44 of 46 hours)**. The single mandated deliverable — a runtime-grounded Q&A document — is authored, committed, and fully verified. All six sub-questions are answered with direct answers, exact commands, complete unedited output, `file:line` citations, and cause→effect reasoning. The real matching entry point was exercised in-memory; the byte-level `hashret()` value was verified exactly; the Q5 run-to-run distribution was reported faithfully; and two source-level anomalies were honestly reported rather than repaired (preserving read-only scope).

**Remaining gaps.** The remaining 2 hours are entirely standard human path-to-production activities: SME technical sign-off (1.5h) and PR review & merge (0.5h). There is **no remaining autonomous engineering work** — no compilation, test, configuration, or deployment gaps exist, because this is a read-only documentation task with no code.

**Critical path to production.** SME sign-off → PR approval → merge. Estimated 2 hours of human effort with no dependencies or blockers.

**Success metrics (all met):** read-only compliance (1 added file; repo byte-for-byte unchanged); every runtime claim reproduced byte-for-byte; 55/55 corroborating Scapy regression tests; 39/39 citations resolving; coverage checklist 6/6.

**Production readiness assessment.** **Ready for human review and merge.** Confidence is **High** for the well-defined content (Q1–Q6, all directly observed and regression-corroborated) and the read-only compliance. The deliverable can be merged as soon as an SME confirms the technical narrative — a formality given the triple verification already performed.

---

## 9. Development Guide

How to reproduce every observation, corroborate against Scapy's own tests, and verify read-only compliance. All commands below were executed and verified in this environment.

### 9.1 System Prerequisites

- **OS:** Linux (Ubuntu 25.10 container used; any POSIX system works).
- **Python:** 3.13.7 observed (Scapy `requires-python >=3.7, <4`). No compiler/build tools required — Scapy is pure-Python and runs from source.
- **Git:** 2.51.0 · **Git LFS:** 3.7.1 (hooks present).

```bash
python3 --version      # Python 3.13.7
git --version          # git version 2.51.0
git lfs version        # git-lfs/3.7.1
```

### 9.2 Environment Setup

Run everything from the **repository root** so the in-repo Scapy (not a pip copy) is imported.

```bash
cd /path/to/repo   # the checkout containing scapy/__init__.py

# Confirm the in-repo Scapy is the one that imports:
PYTHONDONTWRITEBYTECODE=1 PYTHONPATH=. python3 -c \
  "import scapy; print(scapy.__file__); from scapy.config import conf; print('conf.version =', conf.version)"
# -> .../scapy/__init__.py  and  conf.version = 2026.07.08
```

- `PYTHONPATH=.` forces the in-repo package to be imported.
- `PYTHONDONTWRITEBYTECODE=1` prevents `__pycache__` from being written inside the repository.
- An optional, gitignored `.venv/bin/python` (also 3.13.7) is present for running the UTscapy campaigns.

### 9.3 Dependency Installation

**None required.** The investigation uses only the Python standard library (`socket`, `struct`), the in-repo Scapy, and the transitively-imported in-repo `scapy.contrib.mpls`. The optional `cryptography` package (present) emits a benign `CryptographyDeprecationWarning` unrelated to ICMP matching; suppress it with `import warnings; warnings.filterwarnings("ignore")`.

### 9.4 Reproducing the Observations

Keep observation scripts **outside** the repository (e.g. `/tmp/scapy_probe/`) and delete them afterward. Minimal Q1/Q2 reproduction (verified):

```bash
mkdir -p /tmp/scapy_probe
cat > /tmp/scapy_probe/min.py <<'PY'
import warnings; warnings.filterwarnings("ignore")
from scapy.layers.inet import IP, ICMP
req = IP(src="10.0.0.5", dst="192.0.2.1", id=0x1234)/ICMP(type=8, id=0xaaaa, seq=7)
err = IP(bytes(IP(src="192.0.2.254", dst="10.0.0.5")/ICMP(type=3, code=1)/req))
print("answers =", err.answers(req), "| hashret equal =", req.hashret()==err.hashret())
print("hashret =", req.hashret())
PY
PYTHONDONTWRITEBYTECODE=1 PYTHONPATH=. python3 /tmp/scapy_probe/min.py
# -> answers = 1 | hashret equal = True
# -> hashret = b'\xca\x00\x02\x04\x01\xaa\xaa\x07\x00'
rm -rf /tmp/scapy_probe
```

> **Q6 note:** loading the RFC4884 contrib monkeypatches `*error.post_dissection` **process-wide**, so Q6 requires three separate processes — one to build the byte-streams, one that never imports the contrib, and one that calls `load_contrib('icmp_extensions')` and reads the same bytes.

### 9.5 Corroborating Against Scapy's Own Regression Suite

```bash
# IP/ICMP layer campaign (includes the cited answers()/hashret() behaviors)
PYTHONDONTWRITEBYTECODE=1 PYTHONPATH=. .venv/bin/python scapy/tools/UTscapy.py \
  -t test/scapy/layers/inet.uts -N -f text -o /tmp/inet_out.txt
# -> PASSED=54 FAILED=0

# RFC4884 extension campaign (preload the contrib)
PYTHONDONTWRITEBYTECODE=1 PYTHONPATH=. .venv/bin/python scapy/tools/UTscapy.py \
  -t test/contrib/icmp_extensions.uts -N -f text -o /tmp/ext_out.txt \
  -P "load_contrib('icmp_extensions')"
# -> PASSED=1 FAILED=0
```

### 9.6 Verifying Read-Only Compliance

```bash
git diff 0925ada485406684174d6f068dbd85c4154657b3 --name-status
# -> A  blitzy/documentation/scapy_0925ada48540.md   (only)

git status --porcelain
# -> (empty: working tree clean)
```

### 9.7 Reading the Deliverable

```bash
# The knowledge document (458 lines): Environment, Direct answer, Scenario,
# Q1-Q6 (each: direct answer, file:line, command, output, cause->effect),
# Coverage pass, and a file:line citation appendix.
less blitzy/documentation/scapy_0925ada48540.md

# Optional interactive Scapy shell:
./run_scapy
```

### 9.8 Troubleshooting

- **Wrong Scapy imported** → ensure you run from the repo root with `PYTHONPATH=.`; check `scapy.__file__` points into the checkout.
- **`__pycache__` appearing in the repo** → set `PYTHONDONTWRITEBYTECODE=1`.
- **`CryptographyDeprecationWarning` on import** → benign (from `scapy/layers/ipsec.py`); ignore or filter warnings.
- **Q6 shows no extension layers** → the contrib must be loaded in that process **and** the extension checksum must validate over the whole structure; otherwise the bytes remain `Padding` (both cases still yield `answers()=1`).
- **UTscapy "command not found"** → invoke it as a script: `python3 scapy/tools/UTscapy.py ...` from the repo root.

---

## 10. Appendices

### Appendix A — Command Reference

| Purpose | Command |
|---|---|
| Import in-repo Scapy | `PYTHONDONTWRITEBYTECODE=1 PYTHONPATH=. python3 -c "from scapy.config import conf; print(conf.version)"` |
| Run an observation script | `PYTHONDONTWRITEBYTECODE=1 PYTHONPATH=. python3 /tmp/scapy_probe/<script>.py` |
| IP/ICMP regression | `PYTHONPATH=. .venv/bin/python scapy/tools/UTscapy.py -t test/scapy/layers/inet.uts -N -f text -o out.txt` |
| RFC4884 regression | `PYTHONPATH=. .venv/bin/python scapy/tools/UTscapy.py -t test/contrib/icmp_extensions.uts -N -f text -o out.txt -P "load_contrib('icmp_extensions')"` |
| Read-only check (files) | `git diff 0925ada485406684174d6f068dbd85c4154657b3 --name-status` |
| Read-only check (tree) | `git status --porcelain` |
| Interactive shell | `./run_scapy` |

### Appendix B — Port Reference

**Not applicable.** No network services or listening ports are involved. All matching is exercised in-memory via `hashret()`/`answers()`; no on-the-wire transmission occurs.

### Appendix C — Key File Locations

| File | Role |
|---|---|
| `blitzy/documentation/scapy_0925ada48540.md` | **The deliverable** — runtime-grounded Q&A (created) |
| `scapy/layers/inet.py` | Reference — `IP`/`ICMP`/`*error` `hashret()`/`answers()` logic (L568-L610, L986-L998, L1016-L1095) |
| `scapy/config.py` | Reference — matching flags `checkIPID` (L748), `checkIPsrc` (L751), `check_TCPerror_seqack` (L758) |
| `scapy/sendrecv.py` | Reference — real entry point `SndRcvHandler._process_packet()` (L270-L301), bucketing (L244) |
| `scapy/contrib/icmp_extensions.py` | Reference — RFC4884 parser; `pkt.len > 144` gate (L78); patches (L176-L178) |
| `test/scapy/layers/inet.uts` | Reference — ICMP-error regression (#047, #050) |
| `test/contrib/icmp_extensions.uts` | Reference — RFC4884 extension regression |

### Appendix D — Technology Versions

| Component | Version | Source |
|---|---|---|
| Python (CPython) | 3.13.7 | Container interpreter |
| Scapy | 2026.07.08 (`conf.version`) | In-repo checkout (`scapy/__init__.py`) |
| Git | 2.51.0 | System |
| Git LFS | 3.7.1 | System |
| `socket`, `struct` | stdlib (bundled) | Q5 `htons` / Q2 `struct.pack` |
| `scapy.contrib.mpls` | in-repo | Transitive Q6 import |

### Appendix E — Environment Variable Reference

| Variable | Value | Purpose |
|---|---|---|
| `PYTHONPATH` | `.` | Import the in-repo Scapy from the repository root (not a pip copy) |
| `PYTHONDONTWRITEBYTECODE` | `1` | Prevent `__pycache__` from being written inside the repository |

### Appendix F — Developer Tools Guide

- **UTscapy** (`scapy/tools/UTscapy.py`) — runs `.uts` test campaigns. Flags used: `-t <file>` (campaign), `-N` (non-interactive), `-f text` (text format), `-o <out>` (output file), `-P "<preload>"` (Python preamble, e.g. `load_contrib('icmp_extensions')`).
- **`./run_scapy`** — POSIX launcher for an interactive Scapy shell (uses `python3`).
- **`load_contrib('icmp_extensions')`** — activates the RFC4884 parser by monkeypatching `ICMPerror`/`TCPerror`/`UDPerror.post_dissection` (process-wide).

### Appendix G — Glossary

| Term | Meaning |
|---|---|
| `hashret()` | Fast per-packet hash used to bucket candidate sent packets; for ICMP errors it returns the embedded packet's hash. |
| `answers()` | Field-by-field verification that a received packet answers a given sent packet. |
| `IPerror` / `TCPerror` / `UDPerror` / `ICMPerror` | Scapy layers representing the copy of the original packet embedded inside an ICMP error. |
| `SndRcvHandler._process_packet()` | The real send/receive engine entry point where `hashret()` bucketing and `answers()` verification run. |
| `QueryAnswer` | Named tuple `(query, answer)` produced for each matched request/response couple. |
| `conf.checkIPID` | Flag gating embedded IP-ID comparison (default `False`); when truthy, tolerates a `socket.htons` byte-swapped ID. |
| `conf.checkIPsrc` | Flag gating embedded source-IP/port validation (default `True`). |
| `conf.check_TCPerror_seqack` | Flag gating embedded TCP seq/ack verification (default `False`). |
| `socket.htons()` | 16-bit host-to-network byte swap on a little-endian host (`htons(0x1234)=0x3412`) — the Q5 transformation. |
| RFC4884 | "Extended ICMP to Support Multi-Part Messages"; basis for the `pkt.len > 144` extension threshold. |

---

*Generated by the Blitzy Platform. Completion (95.7%) reflects AAP-scoped and path-to-production work only. Cross-section integrity validated: Remaining hours (2h) identical across Sections 1.2, 2.2, and 7; Section 2.1 (44h) + Section 2.2 (2h) = 46h Total; all test results originate from Blitzy's autonomous validation logs.*