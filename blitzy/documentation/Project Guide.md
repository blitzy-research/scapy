
# Blitzy Project Guide — Scapy `raw_packet_cache` Build/Dissect Q&A

## 1. Executive Summary

### 1.1 Project Overview

This project delivers a single, authoritative markdown knowledge-base document that empirically explains Scapy's packet **build/dissect lifecycle** and its byte-caching mechanism (`raw_packet_cache`). It answers four precise questions (Q1–Q4) about a custom protocol layer's field-calculation order and cache behavior — including the explicit determination of whether `copy()` resolves a stale-cache inconsistency (it does not). The audience is Scapy contributors and protocol-layer developers. Business impact is faster, correct debugging of subtle "packet won't rebuild" issues. The task is strictly **read-only**: exactly one documentation file is created and **zero** repository source files are modified, per the governing rule "SWE-AtlasQnA-Repo".

### 1.2 Completion Status

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'pie1':'#5B39F3','pie2':'#FFFFFF','pieStrokeColor':'#B23AF2','pieOuterStrokeColor':'#B23AF2','pieTitleTextSize':'16px','pieSectionTextColor':'#B23AF2'}}}%%
pie showData
    title Completion — 91.9% Complete
    "Completed Work (AI)" : 34
    "Remaining Work" : 3
```

| Metric | Hours |
|--------|-------|
| **Total Hours** | **37.0** |
| **Completed Hours (AI + Manual)** | **34.0** (AI: 34.0 · Manual: 0.0) |
| **Remaining Hours** | **3.0** |
| **Percent Complete** | **91.9%** |

> Completion is computed with the PA1 AAP-scoped methodology: `Completed ÷ (Completed + Remaining) = 34.0 ÷ 37.0 = 91.9%`. Every AAP-scoped autonomous deliverable is complete and independently verified; the remaining 3.0 h is human review/merge (path-to-production) that cannot be performed autonomously.

### 1.3 Key Accomplishments

- ✅ **Deliverable created and committed** — `blitzy/documentation/scapy_0925ada48540.md` (741 lines) at HEAD `2ebea088`.
- ✅ **All four questions answered explicitly** with verbatim, reproducible runtime output — including the explicit "does `copy()` fix it → **No**" sub-part of Q2.
- ✅ **Empirical-first investigation** — 6 temporary probes (Q1, Q1b, Q2, Q3, Q4, smoke) built, run, and captured before writing; then removed.
- ✅ **Every `file:line` citation verified byte-accurate** against source at commit `0925ada4` (`scapy/packet.py`, `scapy/fields.py`, `doc/scapy/build_dissect.rst`, `scapy/layers/inet.py`).
- ✅ **Read-only scope fully honored** — `git diff 0925ada4..HEAD` touches only the deliverable; all `scapy/`, `doc/`, `test/`, and config files are untouched; working tree clean.
- ✅ **Independently re-verified** — Q2 probe reproduced byte-for-byte (`02014141024242` cached; `copy()` no-fix; `clear_cache()` → `02015a5a024242`); Q1 checksum convergence `0xbebe → 0xbeb8` reproduced; `test/fields.uts` passed (exit 0).

### 1.4 Critical Unresolved Issues

| Issue | Impact | Owner | ETA |
|-------|--------|-------|-----|
| _None._ No compilation errors, failing tests, or unresolved defects exist. | — | — | — |

There are **no critical unresolved issues**. The autonomous validation gates all passed and no fixes were outstanding. The only remaining activity is human sign-off (see 1.6).

### 1.5 Access Issues

| System/Resource | Type of Access | Issue Description | Resolution Status | Owner |
|-----------------|----------------|-------------------|-------------------|-------|
| _None_ | — | No access issues identified. The task is self-contained: in-repo Scapy source, local `.venv`, no external services, credentials, or network dependencies. | N/A | — |

**No access issues identified.**

### 1.6 Recommended Next Steps

1. **[High]** Perform SME technical review & sign-off of the deliverable (validate citations, optionally re-run the documented probes, confirm Q1–Q4 and the `copy()`=No conclusion) — 2.0 h.
2. **[Medium]** Review and merge the single-file PR after confirming read-only scope integrity (zero source changes) — 0.5 h.
3. **[Low]** _Optional:_ Index/link the document into the team knowledge base for discoverability (it is intentionally not wired into the Sphinx build) — 0.5 h.

---

## 2. Project Hours Breakdown

### 2.1 Completed Work Detail

All rows are AI-delivered and trace to specific AAP requirements (Q1–Q4, empirical method, citations, coverage, read-only integrity).

| Component | Hours | Description |
|-----------|-------|-------------|
| Source archaeology & lifecycle ground-truth | 6.0 | Read build/dissect path in `scapy/packet.py` (2553 L) + `scapy/fields.py` flags/`PacketListField`/`FieldLenField` + `doc/scapy/build_dissect.rst` to establish authoritative behavior |
| Empirical investigation (6 probes) | 7.0 | Design/run/capture verbatim output for probes Q1, Q1b, Q2, Q3, Q4, smoke; handle `PYTHONPATH` gotcha & fixed-size payloads |
| Q1 answer & reasoning | 2.5 | Deferred convergence; non-explicit clone (`next(iter(self))`); `show2()` re-dissect; captured `0xbebe → 0xbeb8` |
| Q2 answer & reasoning | 2.5 | Two simultaneous states; explicit `copy()`=No proof; `clear_cache()` fix |
| Q3 answer & reasoning | 2.0 | Direct vs nested-field vs nested-payload asymmetry with results table |
| Q4 answer & reasoning | 2.5 | Full lifecycle: cache population, validity gate, invalidation triggers, why payload edits are invisible |
| Background/lifecycle & supporting sections | 3.0 | Build/dissect background, snapshot function, field-type flags, rendering, official corroboration |
| Exact `file:line` citation sourcing & verification | 2.5 | Verify every locator byte-accurate vs HEAD `0925ada4`; build source-reference list |
| Coverage pass, takeaways, honesty notes, assembly | 2.0 | Coverage-pass summary table, practical takeaways, honesty note on observed vs illustrative values, document assembly |
| QA fix cycles | 1.5 | Commits `0c7e09b1` (+40/−15) and `2ebea088` (+1/−1) addressing QA final-gate findings |
| Final autonomous validation | 2.5 | Re-run 6 probes; automated extractor+differ (byte-for-byte); citation re-verify; `test/fields.uts`; markdown & repo-integrity checks |
| **Total** | **34.0** | |

### 2.2 Remaining Work Detail

| Category | Hours | Priority |
|----------|-------|----------|
| [Path-to-production] Human SME technical review & sign-off of deliverable | 2.0 | High |
| [Path-to-production] PR review & merge to target branch | 0.5 | Medium |
| [Path-to-production] (Optional) Index/link doc into team knowledge base | 0.5 | Low |
| **Total** | **3.0** | |

> Cross-check: Section 2.1 (34.0) + Section 2.2 (3.0) = **37.0 Total Hours** (matches Section 1.2). Section 2.2 total (3.0) matches Section 1.2 Remaining and the Section 7 pie chart.

### 2.3 Total Hours & Completion Verification

| Aggregate | Hours | Source |
|-----------|-------|--------|
| Completed (Section 2.1 total) | 34.0 | 11 AI-delivered components |
| Remaining (Section 2.2 total) | 3.0 | 3 path-to-production tasks |
| **Total Project Hours** | **37.0** | 34.0 + 3.0 |
| **Percent Complete** | **91.9%** | 34.0 / 37.0 * 100 |

Formula: `Completion % = Completed / (Completed + Remaining) = 34.0 / 37.0 = 91.9%`. These figures are identical in Sections 1.2, 2.1, 2.2, 7, and 8.

---

## 3. Test Results

All entries below originate from Blitzy's autonomous validation logs for this project and were independently re-executed during this assessment on CPython 3.11.15 / Scapy 2026.07.01.

| Test Category | Framework | Total Tests | Passed | Failed | Coverage % | Notes |
|---------------|-----------|-------------|--------|--------|-----------|-------|
| Empirical reproduction probes | Python 3.11.15 / Scapy 2026.07.01 | 6 | 6 | 0 | N/A (doc) | Probes Q1, Q1b, Q2, Q3, Q4, smoke — all ran successfully; verbatim output captured and re-verified byte-for-byte |
| Regression — `test/fields.uts` (PacketListField) | UTscapy | 137 | 137 | 0 | N/A | Central to task; exit 0, "UTscapy ended successfully"; re-run during this assessment |
| Regression — full Scapy suite (baseline) | UTscapy | 4883 | 4883 | 0 | N/A | Setup-confirmed baseline; **zero** source files changed, so the baseline is unaffected |
| Markdown structural checks | Manual/script | — | Pass | 0 | N/A | 34 balanced code fences (17 blocks), valid tables, no TODO/FIXME/placeholder markers |

**Summary:** 143 discrete pass/fail tests re-executed (6 probes + 137 `fields.uts`), **0 failures**, plus a 4883-test regression baseline that is unaffected by this read-only change. Coverage percentages are not applicable because the deliverable is a markdown document, not shipped code.

---

## 4. Runtime Validation & UI Verification

- ✅ **Operational** — Scapy imports from the in-repo source tree; `scapy.VERSION = 2026.07.01`.
- ✅ **Operational** — Live build/dissect/cache APIs exercised via probes: `do_dissect()` cache population, `self_build()` validity gate, `do_build()` `post_build` gate, `copy()`, `clear_cache()`, `setfieldval()`, `show()`/`show2()`.
- ✅ **Operational** — Q2 two-state behavior reproduced live: `bytes(o)` returns cached `02014141024242` while `show()` renders the modified `ZZ`.
- ✅ **Operational** — Real-layer smoke round-trip `IP()/TCP()`: dissected `IP.len = 40`, `chksum = 0x7ccd`, `TCP.chksum = 0x917c`, cache fields `['flags','options']`.
- ✅ **Operational** — Markdown deliverable is well-formed (balanced fences, valid tables) and renders cleanly.
- ➖ **Not Applicable** — No web/GUI/UI surface exists; this is a library plus a standalone markdown document, so browser/UI verification does not apply.

---

## 5. Compliance & Quality Review

Cross-mapping of AAP deliverables and governing-rule directives to quality benchmarks. Fixes applied during autonomous validation are noted; there are no outstanding items.

| Benchmark / AAP Directive | Status | Progress | Notes |
|---------------------------|--------|----------|-------|
| Deliverable file mandate — `blitzy/documentation/scapy_0925ada48540.md` | ✅ Pass | 100% | 741 lines, committed at HEAD `2ebea088` |
| Q1 answered with verbatim output + citations | ✅ Pass | 100% | `0xbebe → 0xbeb8`; clone idiom `[packet.py:731-732]`; `show2` re-dissect `[packet.py:1486]` |
| Q2 answered incl. explicit "does `copy()` fix it & why" | ✅ Pass | 100% | Answer **No**; `bytes(o.copy()) = 02014141024242`; preserved by `[packet.py:416]` |
| Q3 answered — direct vs nested asymmetry | ✅ Pass | 100% | `05014141024242` / `02094141024242` / `02014141024242` with table |
| Q4 answered — exact mechanism / lifecycle | ✅ Pass | 100% | cache `0102`; `post_build calls 0→1`; invalidation `→ None` |
| Investigate-by-running-first (empirical) | ✅ Pass | 100% | 6 probes built & run before writing; captured verbatim |
| Quote actual observed output verbatim | ✅ Pass | 100% | All byte strings/checksums reproduced exactly, with the command that produced them |
| Be exact & grounded (`file:line` for every claim) | ✅ Pass | 100% | Every locator verified byte-accurate vs HEAD `0925ada4`; zero citation errors |
| Answer every part + coverage pass | ✅ Pass | 100% | Coverage-pass summary table marks all sub-questions YES |
| Read-only scope (zero source edits; temp scripts removed) | ✅ Pass | 100% | `git diff` = only deliverable; `/tmp/scapy_probe` removed; clean tree |
| Honesty (observed vs illustrative values) | ✅ Pass | 100% | Reports real `0xbebe/0xbeb8`, not AAP's illustrative `0xc330/0xc325`, with an explicit note |
| Markdown quality (no placeholders/TODOs) | ✅ Pass | 100% | Balanced fences, valid tables; QA gates passed |
| Human SME sign-off | ⬜ Pending | 0% | Path-to-production gate (Section 1.6, item 1) |

**Fixes applied during autonomous validation:** QA final-gate findings addressed in `0c7e09b1`; Q4 Section-4 verbatim value corrected to `09020304` in `2ebea088`. Final validator required **no** further fixes.

---

## 6. Risk Assessment

| Risk | Category | Severity | Probability | Mitigation | Status |
|------|----------|----------|-------------|------------|--------|
| Citation line-number drift if Scapy source is later refactored | Technical | Low | Medium | Citations pinned to commit `0925ada4` and include identifiers + verbatim code snippets, so they remain locatable even if line numbers move | Mitigated |
| Version-specific behavior (evidence captured on CPython 3.11.15 / Scapy 2026.07.01) | Technical | Low | Low | Doc documents that the `raw_packet_cache` machinery is pure-Python and version-invariant across `requires-python >=3.7,<4` | Mitigated |
| Documentation staleness as Scapy internals evolve | Operational | Low | Low–Medium | Pinned to a specific commit; fully reproducible probes documented for periodic re-verification | Accepted |
| Doc not discoverable (standalone, not in Sphinx build) | Integration | Low | Low | Optional KB-indexing task (Section 2.2, item 3); standalone placement is by AAP design | By design |
| Security exposure | Security | None | N/A | No source code, dependencies, auth, data, network, or PII handling — deliverable is static markdown | N/A |

**No High or Critical risks.** No security-relevant surface exists for a read-only documentation change.

---

## 7. Visual Project Status

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'pie1':'#5B39F3','pie2':'#FFFFFF','pieStrokeColor':'#B23AF2','pieOuterStrokeColor':'#B23AF2','pieSectionTextColor':'#B23AF2'}}}%%
pie showData
    title Project Hours Breakdown (Total 37.0 h)
    "Completed Work" : 34
    "Remaining Work" : 3
```

**Remaining hours by category (Section 2.2 — total 3.0 h):**

| Category | Hours | Priority |
|----------|-------|----------|
| Human SME technical review & sign-off | 2.0 | High |
| PR review & merge | 0.5 | Medium |
| (Optional) KB indexing | 0.5 | Low |

> Integrity: "Remaining Work" = **3.0 h** here equals Section 1.2 Remaining and the Section 2.2 total. "Completed Work" = **34.0 h** equals Section 2.1 total. Colors: Completed = Dark Blue `#5B39F3`, Remaining = White `#FFFFFF`.

---

## 8. Summary & Recommendations

**Achievements.** The project is **91.9% complete** (34.0 of 37.0 hours). Every AAP-scoped autonomous deliverable is finished and independently verified: the answer document exists and is committed; all four questions (Q1–Q4) are answered with verbatim, reproducible evidence and byte-accurate `file:line` citations; the explicit "does `copy()` fix it → No" determination is proven; and the read-only scope is fully honored (only the deliverable was added; the working tree is clean).

**Remaining gaps.** The remaining **3.0 hours** are entirely path-to-production activities that require a human: SME technical sign-off (2.0 h), PR review & merge (0.5 h), and optional knowledge-base indexing (0.5 h). No engineering rework is required — there are zero compilation errors, zero failing tests, and zero unresolved defects.

**Critical path to production.** SME review → merge. Because autonomous validation already passed all gates and this assessment reproduced the key results independently, the review is expected to be a confirmation/sign-off rather than a fix cycle.

**Success metrics.** All four questions answered (coverage-pass table = 100% YES); zero citation errors; probes reproduced byte-for-byte; `test/fields.uts` green; repository integrity intact.

**Production readiness assessment.** The deliverable is **production-ready pending human sign-off**. Confidence is **High** for Q1–Q4 correctness and read-only integrity (directly verified); confidence is **Medium** only regarding future citation drift, which is already mitigated by pinning and snippet-based citations.

---

## 9. Development Guide

Every command below was executed successfully in this environment.

### 9.1 System Prerequisites

- **OS:** Linux (Ubuntu-based container). **Git:** 2.51.0.
- **Interpreter:** CPython **3.11** recommended (highest tested per `tox.ini`); a repo-local `.venv` provides **3.11.15**. The system interpreter is **3.13.7**.
- **Third-party dependencies:** none — Scapy is pure Python with zero mandatory dependencies.

### 9.2 Environment Setup

```bash
cd /tmp/blitzy/scapy/blitzy-5a53a32b-0d96-475e-8e63-ef2367883c3a_897fa9
export REPO="$(pwd)"

# Activate the provided virtual environment (CPython 3.11.15, editable Scapy install)
source .venv/bin/activate        # or invoke .venv/bin/python directly

# Verify interpreter and library
.venv/bin/python --version                              # -> Python 3.11.15
.venv/bin/python -c "import scapy; print(scapy.VERSION)" # -> 2026.07.01
```

If you must recreate the environment:

```bash
python3.11 -m venv .venv
.venv/bin/pip install -e .        # editable install of the in-repo Scapy
```

> **Note (PEP 668):** the system Python is externally-managed. Do **not** modify the repo to install packages; use the `.venv`, or (only if unavoidable) `pip install --break-system-packages`.

### 9.3 View the Deliverable

```bash
wc -l  blitzy/documentation/scapy_0925ada48540.md   # -> 741
grep -nE "^## " blitzy/documentation/scapy_0925ada48540.md   # section list
```

### 9.4 Example Usage — Reproduce the Q2 Cache Behavior

Probes must run with `PYTHONPATH=$REPO` because a script placed under `/tmp` puts *its own* directory on `sys.path`, not the working directory.

```bash
mkdir -p /tmp/scapy_probe
cat > /tmp/scapy_probe/example_q2.py <<'PY'
import warnings; warnings.filterwarnings("ignore")
from scapy.packet import Packet, Raw
from scapy.fields import ByteField, PacketListField

class Inner(Packet):
    name = "Inner"; fields_desc = [ByteField("itype", 0)]
    def extract_padding(self, s): return s[:2], s[2:]

class Outer(Packet):
    name = "Outer"
    fields_desc = [ByteField("ocount", 0),
                   PacketListField("records", [], Inner, count_from=lambda p: p.ocount)]

o = Outer(bytes.fromhex("02014141024242"))
o.records[0].payload = Raw(load=b"ZZ")
print("bytes(o) after nested-payload edit =", bytes(o).hex(), "(cached original)")
print("bytes(o.copy())                    =", bytes(o.copy()).hex(), "(copy does NOT fix)")
o.clear_cache()
print("bytes(o) after clear_cache()       =", bytes(o).hex(), "(rebuilt with edit)")
PY

PYTHONPATH="$REPO" "$REPO/.venv/bin/python" /tmp/scapy_probe/example_q2.py
```

Expected output (verified):

```text
bytes(o) after nested-payload edit = 02014141024242 (cached original)
bytes(o.copy())                    = 02014141024242 (copy does NOT fix)
bytes(o) after clear_cache()       = 02015a5a024242 (rebuilt with edit)
```

### 9.5 Verification

```bash
# Read-only scope integrity (both should show only the deliverable / be empty)
git status --porcelain
git diff --name-only 0925ada4..HEAD     # -> blitzy/documentation/scapy_0925ada48540.md

# Regression test that exercises PacketListField (central to Q2/Q3)
.venv/bin/python -m scapy.tools.UTscapy -t test/fields.uts -N -f text   # -> exit 0, "UTscapy ended successfully"

# Cleanup temporary probe to preserve read-only scope
rm -rf /tmp/scapy_probe
```

### 9.6 Troubleshooting

- **`ModuleNotFoundError: No module named 'scapy'`** when running a `/tmp` probe → prefix with `PYTHONPATH="$REPO"` (script-dir-on-`sys.path` gotcha).
- **`error: externally-managed-environment`** from `pip` → use the `.venv`; never modify repo files to work around it.
- **Citations don't match line numbers** after a future Scapy refactor → citations are pinned to commit `0925ada4` and include code snippets/identifiers; locate by identifier rather than raw line number.

---

## 10. Appendices

### A. Command Reference

| Purpose | Command |
|---------|---------|
| Activate env | `source .venv/bin/activate` |
| Check interpreter | `.venv/bin/python --version` |
| Check Scapy version | `.venv/bin/python -c "import scapy; print(scapy.VERSION)"` |
| Run a probe | `PYTHONPATH="$REPO" .venv/bin/python /tmp/scapy_probe/<probe>.py` |
| Regression (fields) | `.venv/bin/python -m scapy.tools.UTscapy -t test/fields.uts -N -f text` |
| Read-only check | `git diff --name-only 0925ada4..HEAD` |
| Clean-tree check | `git status --porcelain` |

### B. Port Reference

Not applicable — no servers, services, or network ports are involved (pure library + markdown document).

### C. Key File Locations

| Path | Role |
|------|------|
| `blitzy/documentation/scapy_0925ada48540.md` | **Deliverable** (created) |
| `scapy/packet.py` | Reference — build/dissect lifecycle & `raw_packet_cache` |
| `scapy/fields.py` | Reference — field-type flags, `PacketListField`, `FieldLenField` |
| `doc/scapy/build_dissect.rst` | Reference — official build/`post_build` corroboration |
| `scapy/layers/inet.py` | Reference — `IP`/`TCP` for smoke verification |
| `.venv/` | Local CPython 3.11.15 environment (editable Scapy) |

### D. Technology Versions

| Component | Version | Source |
|-----------|---------|--------|
| CPython (venv, used) | 3.11.15 | repo-local `.venv` (matches `tox.ini` `py311`) |
| CPython (system) | 3.13.7 | container default |
| Scapy | 2026.07.01 | in-repo source (`scapy/__init__.py`) |
| Git | 2.51.0 | container |
| `requires-python` | `>=3.7, <4` | `pyproject.toml:17` |

### E. Environment Variable Reference

| Variable | Value / Example | Purpose |
|----------|-----------------|---------|
| `REPO` | repository root path | Base for `PYTHONPATH` and commands |
| `PYTHONPATH` | `$REPO` | Ensures the in-repo Scapy is imported by `/tmp` probes |

### F. Developer Tools Guide

- **UTscapy** — Scapy's `.uts` unit-test runner: `python -m scapy.tools.UTscapy -t <file>.uts -N -f text`.
- **git** — `git diff --name-only 0925ada4..HEAD` and `git status --porcelain` confirm read-only integrity.
- **venv** — repo-local `.venv` provides the tested CPython 3.11.15 with an editable Scapy install (`__editable__.scapy-2026.7.1.pth`).

### G. Glossary

| Term | Meaning |
|------|---------|
| `raw_packet_cache` | Bytes of a layer captured during `do_dissect()`, returned verbatim by `self_build()` when no tracked field changed |
| `raw_packet_cache_fields` | Per-field change-detection snapshot; for a `PacketListField` it stores only each sub-packet's `.fields` (not payloads) |
| `post_build()` | Hook for late-evaluated fields (checksums/lengths); runs only when the cache is `None` |
| `explicit` | Flag distinguishing dissected packets (`1`) from hand-built ones; non-explicit builds operate on a clone |
| `show()` vs `show2()` | `show()` renders the live tree; `show2()` builds then re-dissects before rendering |
| Probe | A temporary observation script (run from `/tmp`, then removed) used to capture verbatim runtime behavior |
