# Blitzy Project Guide — Scapy Code-Grounded Investigation (commit `0925ada48540`)

> Deliverable under assessment: **`blitzy/documentation/scapy_0925ada48540.md`** — a single,
> comprehensive, code-grounded Markdown document answering five investigative questions about the
> Scapy packet-manipulation library at the pinned commit
> `0925ada485406684174d6f068dbd85c4154657b3`. The entire Scapy source tree was treated as
> **read-only** per the governing user constraint.

---

## 1. Executive Summary

### 1.1 Project Overview

This project is an **investigative, code-grounded Q&A documentation** task — not a feature or
bug-fix effort. The objective was to author one Markdown document that answers five questions about
the Scapy library at a pinned commit: (Q1) shell startup and the reported version, (Q2) ICMP-echo
construction and auto-populated IP fields with `show()` output, (Q3) sending a crafted ICMP packet to
localhost and the network-layer behavior, (Q4) a source-level explanation of default IP construction,
and (Q5) the test-suite count and pass/fail summary. The audience is an engineer onboarding to Scapy
before packet crafting. Every answer was produced by **building and running the actual source** in the
authoritative Docker environment — never by assumption — and the Scapy source tree was never modified.

### 1.2 Completion Status

The completion percentage is computed using the AAP-scoped, hours-based methodology: all autonomous
deliverable work is complete and validated; the only remaining work is the path-to-production human
review and acceptance gate.

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'pie1':'#5B39F3','pie2':'#FFFFFF','pieStrokeColor':'#B23AF2','pieOuterStrokeWidth':'2px','pieTitleTextSize':'16px','pieSectionTextSize':'14px','pieStrokeWidth':'2px'}}}%%
pie showData title Completion Status — 91.2% Complete
    "Completed Work (AI)" : 31
    "Remaining Work" : 3
```

| Metric | Hours |
|--------|-------|
| **Total Hours** | **34.0** |
| Completed Hours (AI + Manual) | 31.0 (AI: 31.0, Manual: 0.0) |
| Remaining Hours | 3.0 |
| **Percent Complete** | **91.2%** |

> Completion formula: `31.0 / (31.0 + 3.0) = 31.0 / 34.0 = 91.2%`.
> Color legend: **Completed = Dark Blue `#5B39F3`**, **Remaining = White `#FFFFFF`**.

### 1.3 Key Accomplishments

- ✅ Single deliverable created at the correct path and filename: `blitzy/documentation/scapy_0925ada48540.md` (772 lines, ~49 KB, 6,431 words).
- ✅ **Q1** answered — live shell-startup transcript captured (PyX INFO line, TripleDES deprecation warnings, ASCII banner, `Welcome to Scapy`, `Version 2.5.0.dev87`, `using IPython 9.4.0`, `>>>` prompt) with the `_version()` four-method fallback chain explained.
- ✅ **Q2** answered — `IP()/ICMP()` constructed; `show()` pre-build dump captured (`ihl`/`len`/`chksum` deferred as `None`); exact 28-byte serialization and a byte-offset→field map provided.
- ✅ **Q3** answered — `send(IP(dst="127.0.0.1")/ICMP())` emits `Sent 1 packets.`; L3-socket selection, loopback resolution (`lo`), and emission accounting traced through source.
- ✅ **Q4** answered — declarative `fields_desc` defaults and the `None`-sentinel `post_build` resolution explained, situated in the generic build lifecycle.
- ✅ **Q5** answered — corpus counted two independent ways (**193 `.uts` files / 5,318 unit tests**; `linux.utsc` selects **190 / 5,274**); authoritative UTscapy campaign pass/fail summary transcribed.
- ✅ **122 `file:line` citations** across 17 source files, all verified accurate against the pinned-commit source (zero inaccurate citations).
- ✅ Source-tree integrity preserved — **zero** source files modified; working tree clean; temporary probe scripts created only in `/tmp` and deleted.
- ✅ Autonomous validation confirmed **100% factual accuracy** across all five questions with **zero discrepancies**; no fixes were required.

### 1.4 Critical Unresolved Issues

| Issue | Impact | Owner | ETA |
|-------|--------|-------|-----|
| _None_ — no compilation errors, no failing in-scope checks, no missing content | None | — | — |

> There are no critical unresolved issues. The deliverable is factually accurate, internally
> consistent, and well-formed. The `test/cert.uts` early-halt observed during Q5 is an
> **environment-only** condition (a `cryptography` API removal) that the AAP explicitly places **out of
> scope** and that the document correctly characterizes as distinct from any Scapy code defect — it is
> not an unresolved issue of this deliverable.

### 1.5 Access Issues

| System/Resource | Type of Access | Issue Description | Resolution Status | Owner |
|-----------------|----------------|-------------------|-------------------|-------|
| Authoritative Docker image (`ghcr.io/scaleapi/swe-atlas:swe_atlas_QnA_secdev_scapy_1.0`) | Container registry pull | Required to reproduce authoritative values (version `2.5.0.dev87`, test summary) | Resolved — image was available and used during validation | Reviewer |
| Raw-socket transmission (Q3) | OS capability (`CAP_NET_RAW` / root) | Sending to `127.0.0.1` requires elevated privileges; satisfied in Docker (uid 0) | Resolved — root available in authoritative image | Reviewer |

> No blocking access issues were identified. Both items above were satisfied in the authoritative
> environment during autonomous validation; they are listed for reviewer awareness when reproducing.

### 1.6 Recommended Next Steps

1. **[High]** Perform a subject-matter-expert technical-accuracy review of `blitzy/documentation/scapy_0925ada48540.md` — read through and spot-check a sample of the 122 citations against the pinned-commit source.
2. **[Medium]** Review the pull request, confirm `git diff` shows exactly one added file (source tree untouched), then approve and merge into the destination repository.
3. **[Low]** Verify the document renders correctly in the destination Markdown viewer (tables, code fences, inline link).

---

## 2. Project Hours Breakdown

### 2.1 Completed Work Detail

All completed work was performed autonomously (AI). Each component traces to a specific AAP requirement.

| Component | Hours | Description |
|-----------|-------|-------------|
| Environment setup & dual-environment analysis | 2.5 | Pull/verify authoritative Docker image, confirm pinned commit, inventory dependencies, build the authoritative-vs-fallback environment table (AAP §0.6, Section 0 of deliverable) |
| Q1 — Shell startup + version resolution | 3.5 | Launch shell, capture full banner transcript, trace `_version()` four-method fallback, derive `2.5.0.dev87` from `git describe` (`v2.5.0-87-g0925ada4`), explain `_parse_tag` regex |
| Q2 — ICMP echo, `show()` & byte-level analysis | 3.5 | Construct `IP()/ICMP()`, capture `show()`/`repr`/raw bytes, build 13-row byte-offset→field map, re-dissect, explain `proto`/`bind_layers` special case |
| Q3 — Localhost send + network-layer path | 2.5 | Run `send()`, capture `conf.L3socket`/`loopback_name`/route tuple, trace `send → _send → __gen_send`, document emission accounting & privilege note |
| Q4 — Default IP construction source analysis | 3.0 | Read & explain `fields_desc` defaults, `None`-sentinel `post_build` resolution, and the generic build lifecycle; embed accurate source blocks |
| Q5 — Test suite execution, count & diagnosis | 4.0 | Count `.uts` two ways (193/5,318), replicate `linux.utsc` glob+remove logic (190/5,274), run canonical & individual UTscapy campaigns, diagnose the `cert.uts` halt, separate environment-only failures |
| Code citation tracing & verification | 2.5 | Locate, verify and format 122 `file:line` citations across 17 source files against pinned-commit source |
| Rationale authoring | 2.0 | Compose the "why" narrative for all five questions (rule-mandated rationale) |
| Document assembly, structure & closing note | 2.5 | Intro, consistent Answer/Evidence/Citation/Rationale templating, Markdown tables & fences, closing note, unifying theme, correct path/filename |
| Read-only discipline & temp-script cleanup | 1.0 | Run all probes under `/tmp`, delete 8 scripts, verify `git status` clean throughout |
| Autonomous validation & fact-check (Final Validator) | 4.0 | 12-phase validation: fact-check 100% of claims/transcripts/counts/citations vs Docker runs, replicate counts, run full suite |
| **Total Completed** | **31.0** | |

### 2.2 Remaining Work Detail

All remaining work is the path-to-production human review and acceptance gate.

| Category | Hours | Priority |
|----------|-------|----------|
| SME technical-accuracy review & sign-off of the deliverable | 2.0 | High |
| PR review & merge approval into destination repository | 0.5 | Medium |
| Render verification in destination Markdown viewer | 0.5 | Low |
| **Total Remaining** | **3.0** | |

### 2.3 Hours Reconciliation

| Quantity | Hours |
|----------|-------|
| Section 2.1 Completed total | 31.0 |
| Section 2.2 Remaining total | 3.0 |
| **Total Project Hours (2.1 + 2.2)** | **34.0** |
| Percent Complete (31.0 / 34.0) | 91.2% |

> Cross-section check: Section 2.1 (31.0) + Section 2.2 (3.0) = 34.0 = Total in Section 1.2 ✓.
> Remaining (3.0) is identical in Sections 1.2, 2.2, the Section 4 task list, and the Section 7 pie chart ✓.

---

## 3. Test Results

All tests below originate from **Blitzy's autonomous validation logs** for this project. Because the
deliverable is documentation, "tests" comprise (a) the **UTscapy campaign executions** run in the
authoritative Docker image to ground the Q5 answer, and (b) the **claim-by-claim fact-check** of the
document against real Docker runs (the true acceptance test for a documentation deliverable). All
campaign values below are the authoritative Docker results (Python 3.11.13, Scapy `2.5.0.dev87`),
run in non-root campaign mode (`-N`).

| Test Category | Framework | Total Tests | Passed | Failed | Coverage % | Notes |
|---------------|-----------|-------------|--------|--------|-----------|-------|
| Core / RNG (`test/random.uts`) | UTscapy | 11 | 11 | 0 | 100% | Clean in authoritative image |
| Field types (`test/fields.uts`) | UTscapy | 138 | 138 | 0 | 100% | Clean in authoritative image |
| DNS layer (`test/scapy/layers/dns.uts`) | UTscapy | 18 | 18 | 0 | 100% | Clean in authoritative image |
| IPv4/INET layer (`test/scapy/layers/inet.uts`) | UTscapy | 54 | 54 | 0 | 100% | Clean in authoritative image (locally 52/2 due to missing `mock`) |
| DHCP layer (`test/scapy/layers/dhcp.uts`) | UTscapy | 7 | 7 | 0 | 100% | Clean in authoritative image (locally 6/1 due to missing `IPython`) |
| Answering machines (`test/answering_machines.uts`) | UTscapy | 11 | 11 | 0 | 100% | Clean in authoritative image (`mock`/`IPython` present) |
| TLS netaccess (`test/tls/tests_tls_netaccess.uts`) | UTscapy | 0 | 0 | 0 | n/a | Skipped under `-N` (non-root) by design |
| TLS cert (`test/cert.uts`) | UTscapy | 63 | 1 | 62 | n/a | **Environment-only** halt — `cryptography 45.0.6` removed `hazmat.backends.openssl.ec` (`cert.py:L51`); AAP-out-of-scope to fix |
| Document claim fact-check (all 5 Q's) | Blitzy autonomous validation | 5 | 5 | 0 | 100% | Every claim/transcript/count/citation matched Docker reality — zero discrepancies |
| Citation accuracy verification | Blitzy autonomous validation | 122 | 122 | 0 | 100% | All 122 `file:line` locators verified accurate (17 files) |

**Corpus scope (environment-independent, verified two independent ways):** the test tree contains
**193 `.uts` campaign files** holding **5,318 unit tests**; the Linux campaign config (`linux.utsc`)
selects **190 files / 5,274 tests** (excluding `test/windows.uts` and `test/bpf.uts`).

> Integrity note: the individual campaign rows above are representative authoritative campaigns
> executed during validation, not the entire 5,274-test selection; the full canonical run halts early
> at `test/cert.uts` because `linux.utsc` sets `breakfailed: true`. The `cert.uts` failure is an
> environment-only `cryptography` incompatibility (out of scope per AAP §0.5.2), not a Scapy code
> defect, and the document states this explicitly.

---

## 4. Runtime Validation & UI Verification

This is a terminal/library project — **no web UI, database, or HTTP server**. "Runtime validation"
means the Scapy shell, packet crafting, localhost send, and the UTscapy harness were actually executed
in the authoritative Docker image.

- ✅ **Operational** — Scapy interactive shell launches from source (`PYTHONPATH=. python3 -m scapy`); banner and `Version 2.5.0.dev87` rendered; IPython 9.4.0 REPL active.
- ✅ **Operational** — `scapy.VERSION == scapy.__version__ == conf.version == '2.5.0.dev87'` confirmed by import in the authoritative image.
- ✅ **Operational** — `IP()/ICMP()` constructs; `show()` renders the field dump; `raw(...)` serializes to the exact 28-byte datagram (`4500001c0001000040017cde7f0000017f0000010800f7ff00000000`).
- ✅ **Operational** — `send(IP(dst="127.0.0.1")/ICMP())` emits over `lo` and prints `Sent 1 packets.` (root/`CAP_NET_RAW` available in Docker).
- ✅ **Operational** — UTscapy harness runs; per-campaign `PASSED=/FAILED=` summaries captured; core campaigns pass cleanly.
- ⚠ **Partial** — Full canonical UTscapy run halts early at `test/cert.uts` (`PASSED=1 FAILED=62`) due to an environment-only `cryptography` API removal; out of scope to fix per AAP §0.5.2 and documented as such.
- ✅ **Operational (document artifact)** — Markdown is well-formed: 26 balanced code fences, 27 table rows, 1 valid inline link, no TODO/FIXME/placeholder markers.
- ➖ **N/A** — No GUI, no responsive breakpoints, no browser/Lighthouse verification applicable (terminal/library project).

---

## 5. Compliance & Quality Review

Cross-mapping of AAP deliverables and governing rules (rule set **"SWE-AtlasQnA-Repo"**) to outcomes.

| Requirement / Rule | Source | Status | Progress | Notes |
|--------------------|--------|--------|----------|-------|
| Single Markdown document named `<source_branch>.md` | AAP §0.7 | ✅ Pass | 100% | `scapy_0925ada48540.md` created |
| Placed in `blitzy/documentation/` | AAP §0.7 | ✅ Pass | 100% | Correct destination path |
| Q1 — startup + version, with rationale | AAP §0.1.1 | ✅ Pass | 100% | Live transcript + `_version()` fallback explained |
| Q2 — ICMP echo, auto-populated fields, `show()` | AAP §0.1.1 | ✅ Pass | 100% | `show()` dump + 28-byte map captured |
| Q3 — localhost send, network-layer behavior | AAP §0.1.1 | ✅ Pass | 100% | Emission path traced; `Sent 1 packets.` |
| Q4 — default IP construction (source-level) | AAP §0.1.1 | ✅ Pass | 100% | `fields_desc` + `post_build` explained |
| Q5 — test count + pass/fail summary | AAP §0.1.1 | ✅ Pass | 100% | 193/5,318 corpus; campaign summary transcribed |
| Build-and-run for evidence (no fabrication) | AAP §0.7 | ✅ Pass | 100% | All values from real Docker runs |
| Code is source of truth | AAP §0.7 | ✅ Pass | 100% | 122 verified `file:line` citations |
| Rationale included per answer | AAP §0.7 | ✅ Pass | 100% | Dedicated Rationale subsection per question |
| Do not modify source files | AAP §0.7 / user constraint | ✅ Pass | 100% | `git diff` = 1 file added, 0 source touched |
| No other code added to source repo | AAP §0.7 | ✅ Pass | 100% | Only the one document |
| Temp scripts cleaned up | User constraint | ✅ Pass | 100% | 8 `/tmp` probe scripts deleted |
| Working tree verified clean | AAP §0.8 | ✅ Pass | 100% | `git status --porcelain` empty |
| Markdown well-formedness | Quality | ✅ Pass | 100% | Balanced fences, valid tables/link, no placeholders |

**Fixes applied during autonomous validation:** none were required — validation confirmed the existing
deliverable was already 100% factually accurate and internally consistent.
**Outstanding compliance items:** none. All rules satisfied.

---

## 6. Risk Assessment

Overall risk posture is **Low** — the deliverable adds no code, modifies no source, and was validated
at 100% accuracy.

| Risk | Category | Severity | Probability | Mitigation | Status |
|------|----------|----------|-------------|------------|--------|
| Environment divergence — version & test values differ outside the authoritative Docker image (e.g., local Python 3.13, 0 git tags → date-string version) | Technical | Low | Medium | Document explicitly labels every authoritative value vs. the non-authoritative "local fallback" | Mitigated |
| `test/cert.uts` early-halt — `cryptography 45.0.6` removed `hazmat.backends.openssl.ec` (`cert.py:L51`) | Technical | Low | Low | Documented as environment-only; AAP §0.5.2 places remediation out of scope; distinguished from code defects | Documented / Out-of-scope |
| Raw-socket privilege — Q3 send requires root/`CAP_NET_RAW`; without it a `PermissionError` is raised | Security | Low | Low | Privilege requirement documented; behavior to be reported as observed, never worked around | Documented |
| Documentation drift — 122 `file:line` citations are pinned to commit `0925ada4` and could drift over time | Operational | Low | Low | Citations explicitly anchored to the pinned commit; commit is immutable | Accepted |
| Render fidelity — 27 tables / 26 code fences may render differently in the destination viewer | Operational | Low | Low | Covered by the Low-priority render-verification human task (HT-3) | Open |
| Integration risk — none material; the document introduces no code, imports, APIs, or external service dependencies | Integration | Low | Low | Self-contained document; no coupling | N/A |

---

## 7. Visual Project Status

```mermaid
%%{init: {'theme':'base', 'themeVariables': {'pie1':'#5B39F3','pie2':'#FFFFFF','pieStrokeColor':'#B23AF2','pieOuterStrokeWidth':'2px','pieTitleTextSize':'16px','pieSectionTextSize':'14px','pieStrokeWidth':'2px'}}}%%
pie showData title Project Hours Breakdown (Total 34h)
    "Completed Work" : 31
    "Remaining Work" : 3
```

**Remaining hours by category (Section 2.2 — priority distribution):**

| Category | Hours | Priority |
|----------|-------|----------|
| SME accuracy review | 2.0 | High |
| PR review & merge | 0.5 | Medium |
| Render verification | 0.5 | Low |
| **Total** | **3.0** | |

> Color legend: **Completed = Dark Blue `#5B39F3`**, **Remaining = White `#FFFFFF`** (headings/accents
> Violet-Black `#B23AF2`, highlight Mint `#A8FDD9`).
> Integrity: "Remaining Work" = **3** here = Section 1.2 Remaining (3.0) = sum of Section 2.2 Hours
> (2.0 + 0.5 + 0.5 = 3.0) ✓.

---

## 8. Summary & Recommendations

**Achievements.** The project delivered a single, comprehensive, code-grounded document
(`blitzy/documentation/scapy_0925ada48540.md`, 772 lines) that answers all five investigative questions
about Scapy at the pinned commit. Every transcript, byte dump, field value, version string, and test
count was produced by actually building and running the source in the authoritative Docker image, and
each answer is accompanied by precise source citations (122 `file:line` locators across 17 files) and
explicit rationale. Autonomous validation confirmed 100% factual accuracy with zero discrepancies and
required no fixes.

**Remaining gaps.** The project is **91.2% complete** (31.0 of 34.0 hours). The remaining 3.0 hours are
entirely the **path-to-production human acceptance gate**: a subject-matter-expert accuracy review
(2.0h), PR review and merge (0.5h), and render verification in the destination viewer (0.5h). There is
no remaining engineering work — no compilation errors, no failing in-scope checks, and no missing
content.

**Critical path to production.** SME accuracy review → PR approval and merge → optional render check.
No blockers exist on this path.

**Success metrics.** All five questions answered with evidence and rationale (5/5); source tree
unmodified (0 files changed); all governing rules satisfied (15/15 in the compliance matrix); document
well-formed (26 balanced fences, 27 tables, 0 placeholders).

**Production-readiness assessment.** The deliverable is **production-ready pending human sign-off**. The
only "failing" observation in the entire project — the `test/cert.uts` UTscapy halt — is an
environment-only `cryptography` incompatibility that the AAP explicitly excludes from scope and that the
document accurately characterizes. Recommendation: **approve and merge after the SME review.**

| Metric | Value |
|--------|-------|
| Completion | 91.2% (31.0 / 34.0 h) |
| Questions answered | 5 / 5 |
| Source files modified | 0 |
| Compliance items passed | 15 / 15 |
| Citations verified | 122 / 122 |
| Critical unresolved issues | 0 |

---

## 9. Development Guide

This guide explains how to reproduce the investigation environment, run the same evidence-gathering
commands the deliverable used, and review the document.

### 9.1 System Prerequisites

- **Authoritative path (recommended):** Docker Engine (to run the pinned image that reproduces the
  exact reported values).
- **Reproduction-only path:** a POSIX shell, `git`, and Python (`requires-python = ">=3.7, <4"`,
  `pyproject.toml:L17`; the authoritative image uses **Python 3.11.13**).
- On Linux/BSD, Scapy has **no mandatory external Python dependencies** and runs directly from a source
  checkout.

### 9.2 Environment Setup

**Option A — Authoritative Docker image (reproduces `2.5.0.dev87` and the canonical test summary):**

```bash
# Image checks out /app at the pinned commit, runs as root, ships Python 3.11 + mock + IPython + cryptography
docker pull ghcr.io/scaleapi/swe-atlas:swe_atlas_QnA_secdev_scapy_1.0
IMG=ghcr.io/scaleapi/swe-atlas:swe_atlas_QnA_secdev_scapy_1.0

# Confirm the pinned commit
docker run --rm "$IMG" -lc 'cd /app && git rev-parse HEAD'
# Expected: 0925ada485406684174d6f068dbd85c4154657b3
```

**Option B — Local source checkout (reproduction only; values diverge as noted in Troubleshooting):**

```bash
git clone https://github.com/secdev/scapy.git
cd scapy
git checkout 0925ada485406684174d6f068dbd85c4154657b3
```

### 9.3 Dependency Installation

```bash
# Running from source on Linux requires NO installation.
# Optional extras (enhanced REPL, dumps, crypto layers):
python3 -m pip install --user "ipython"           # enhanced shell (Q1 banner shows "using IPython")
python3 -m pip install --user "pyx" "matplotlib"  # psdump()/pdfdump() and plotting (optional)
python3 -m pip install --user "cryptography"      # TLS/IPsec layers (optional)
```

### 9.4 Application Startup

```bash
# Start the Scapy shell from source (equivalent to the run_scapy launcher: PYTHONPATH=$DIR python3 -m scapy)
PYTHONPATH=. python3 -m scapy
#   ...or:
./run_scapy
```

### 9.5 Verification Steps

```bash
# Verify the reported version (authoritative Docker => 2.5.0.dev87)
docker run --rm "$IMG" -lc 'cd /app && python3 -c "import scapy; print(scapy.VERSION)"'

# Verify the deliverable exists and is well-formed
wc -l blitzy/documentation/scapy_0925ada48540.md          # => 772
grep -c "^\`\`\`" blitzy/documentation/scapy_0925ada48540.md  # => 26 (even = balanced)

# Verify source tree was untouched (should print exactly one added file)
git diff --name-status 0925ada485406684174d6f068dbd85c4154657b3..HEAD
# => A   blitzy/documentation/scapy_0925ada48540.md
git status --porcelain    # => empty (clean tree)
```

### 9.6 Example Usage (the five investigation commands)

```bash
IMG=ghcr.io/scaleapi/swe-atlas:swe_atlas_QnA_secdev_scapy_1.0

# Q1 — shell startup + version banner
docker run --rm "$IMG" -lc 'cd /app && echo exit | python3 -m scapy'

# Q2 — ICMP echo construction + show()
docker run --rm "$IMG" -lc "cd /app && python3 -c \"from scapy.all import *; (IP()/ICMP()).show()\""

# Q3 — send a crafted ICMP packet to localhost
docker run --rm "$IMG" -lc "cd /app && python3 -c \"from scapy.all import *; send(IP(dst='127.0.0.1')/ICMP())\""

# Q5 — run the UTscapy suite (Linux campaign, non-root mode)
docker run --rm "$IMG" -lc 'cd /app && python3 -u -m scapy.tools.UTscapy -c ./test/configs/linux.utsc -N'

# Corpus counts (environment-independent; runnable anywhere in the checkout)
find test -name "*.uts" | wc -l                                 # => 193 files
find test -name "*.uts" -exec grep -h "^=" {} \; | wc -l         # => 5318 unit tests
```

### 9.7 Troubleshooting

- **Version shows a date like `2026.06.26` instead of `2.5.0.dev87`.** Expected outside the
  authoritative image: a tag-less clone (0 git tags) forces `_version()` past `git describe` to its
  file-mtime fallback. With the image's 533 tags, `git describe` yields `v2.5.0-87-g0925ada4` →
  `2.5.0.dev87`. This is by design, not a defect.
- **UTscapy halts at `test/cert.uts` (`PASSED=1 FAILED=62`).** Caused by `cryptography 45.0.6` removing
  `cryptography.hazmat.backends.openssl.ec` imported at `scapy/layers/tls/cert.py:L51`. This is an
  **environment-only** issue and is **out of scope** to fix per AAP §0.5.2 — it is not a Scapy code
  defect. To run only the healthy core campaigns, target individual files, e.g.
  `python3 -m scapy.tools.UTscapy -t test/fields.uts -N`.
- **`PermissionError: [Errno 1] Operation not permitted` on send.** Raw-socket transmission needs
  root or the `CAP_NET_RAW` capability; the authoritative image runs as root. Report the error as
  observed behavior — do not modify source to work around it.
- **`INFO: Can't import PyX...` on startup.** Benign — PyX is optional (used only by
  `psdump()`/`pdfdump()`).
- **Local test failures referencing `mock` or `IPython`.** Test-only dependencies absent in a bare
  local environment but present in the authoritative image; the authoritative campaign values are the
  ground truth.

---

## 10. Appendices

### Appendix A — Command Reference

| Purpose | Command |
|---------|---------|
| Confirm pinned commit (Docker) | `docker run --rm "$IMG" -lc 'cd /app && git rev-parse HEAD'` |
| Start Scapy shell from source | `PYTHONPATH=. python3 -m scapy` (or `./run_scapy`) |
| Print version | `python3 -c "import scapy; print(scapy.VERSION)"` |
| Construct + `show()` | `python3 -c "from scapy.all import *; (IP()/ICMP()).show()"` |
| Send to localhost | `python3 -c "from scapy.all import *; send(IP(dst='127.0.0.1')/ICMP())"` |
| Run UTscapy (Linux campaign) | `python3 -u -m scapy.tools.UTscapy -c ./test/configs/linux.utsc -N` |
| Run a single campaign | `python3 -m scapy.tools.UTscapy -t test/fields.uts -N` |
| Count `.uts` files | `find test -name "*.uts" \| wc -l` |
| Count unit tests | `find test -name "*.uts" -exec grep -h "^=" {} \; \| wc -l` |
| Verify source untouched | `git diff --name-status 0925ada4..HEAD` |
| Clean-tree check | `git status --porcelain` |

### Appendix B — Port Reference

Not applicable. Scapy is a terminal/library project with no listening services. Q3 transmits over the
loopback interface `lo` using a raw/`PF_PACKET` socket (no TCP/UDP port is bound).

### Appendix C — Key File Locations

| Path | Role |
|------|------|
| `blitzy/documentation/scapy_0925ada48540.md` | **The deliverable** (only file added) |
| `scapy/__init__.py` | `_version()` resolver (Q1) — `:L122-167` |
| `scapy/main.py` | `interact()` + banner (Q1) — `:L503,L595,L632-639,L657` |
| `scapy/layers/inet.py` | `IP`/`ICMP` `fields_desc` + `post_build` (Q2/Q4) — `:L521-551,L952-979` |
| `scapy/packet.py` | build lifecycle + `show()` (Q2/Q4) — `:L678,L746,L758,L1459` |
| `scapy/sendrecv.py` | `send()`/`__gen_send()` (Q3) — `:L332,L380,L392,L422` |
| `scapy/config.py` | `conf.L3socket`, `loopback_name` (Q3) — `:L647,L672,L911` |
| `scapy/arch/linux.py` | Linux loopback/route (Q3) — `:L243-388` |
| `scapy/tools/UTscapy.py` | UTscapy runner (Q5) — `:L619,L622` |
| `test/configs/linux.utsc` | Linux campaign config (Q5) |
| `run_scapy`, `pyproject.toml`, `tox.ini`, `README.md` | Launchers/metadata (setup) |

### Appendix D — Technology Versions

| Component | Authoritative (Docker) | Local fallback (non-authoritative) |
|-----------|------------------------|-------------------------------------|
| Python | 3.11.13 | 3.13.7 |
| Reported Scapy version | `2.5.0.dev87` (`git describe`; 533 tags) | `2026.06.26` (file-mtime; 0 tags) |
| `IPython` | 9.4.0 (present) | absent |
| `mock` (test-only) | 5.2.0 (present) | absent |
| `cryptography` | 45.0.6 (present) | varies |
| `python-can` (test-only) | 4.6.1 (present) | absent |
| `PyX` / `matplotlib` | absent (optional) | absent |

### Appendix E — Environment Variable Reference

| Variable | Purpose |
|----------|---------|
| `PYTHONPATH=.` | Run Scapy from a source checkout without installation (used by `run_scapy`) |
| `PYTHON` | Override the interpreter `run_scapy` invokes (defaults to `python3`) |
| `SCAPY_VERSION` | First branch of `_version()`; if set, overrides the resolved version (unset here) |

### Appendix F — Developer Tools Guide

- **UTscapy** (`scapy/tools/UTscapy.py`) — Scapy's bespoke `.uts` test harness (not pytest/unittest).
  Key flags: `-c <config.utsc>` (campaign config), `-t <file.uts>` (single campaign), `-N` (non-root
  mode), `-K <keyword>` / `-k <keyword>` (filter). A unit test is any `.uts` line beginning with `=`.
- **`tox`** orchestrates per-platform `.utsc` configs; the canonical Linux invocation is
  `tox.ini:L42` → `-m scapy.tools.UTscapy -c ./test/configs/linux.utsc -N`.
- **`run_scapy`** — source launcher: `PYTHONPATH=$DIR exec "$PYTHON" -m scapy "$@"`.

### Appendix G — Glossary

| Term | Meaning |
|------|---------|
| AAP | Agent Action Plan — the authoritative project requirements |
| `fields_desc` | Scapy's per-layer declarative field list defining defaults |
| `None`-sentinel | A field default of `None` signaling "compute me at build time" (e.g., `len`, `chksum`, `ihl`) |
| `post_build` | Per-layer hook that resolves deferred fields during serialization |
| UTscapy | Scapy's custom unit-test framework driving `.uts` campaign files |
| `.uts` | A UTscapy campaign file; each line starting with `=` defines one unit test |
| `breakfailed` | `linux.utsc` setting that halts the run at the first failing campaign |
| L3 send | A layer-3 transmission where Scapy supplies the link layer itself |
| `2.5.0.dev87` | Authoritative version: 87 commits past the `v2.5.0` tag (via `git describe`) |