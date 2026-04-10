# Blitzy Project Guide

## 1. Executive Summary

### 1.1 Project Overview

This project delivers a comprehensive technical deep-dive document investigating Scapy's packet field calculation order, raw packet caching mechanism, cache invalidation rules, and the "dual state" behavior observed when modifying nested payloads within dissected packets. The target audience is developers working with custom Scapy protocol layers who encounter confusing rebuild behavior. The deliverable is a single new Markdown file (`blitzy/documentation/scapy_0925ada48540.md`, 697 lines) that answers six specific technical questions, all grounded in source code line-number citations and validated with empirical tests. No existing source files were modified.

### 1.2 Completion Status

```mermaid
pie title Project Completion
    "Completed (AI)" : 21
    "Remaining" : 1.5
```

| Metric | Value |
|--------|-------|
| **Total Project Hours** | 22.5 |
| **Completed Hours (AI)** | 21 |
| **Remaining Hours** | 1.5 |
| **Completion Percentage** | 93.3% |

**Calculation**: 21 completed hours / (21 + 1.5 remaining hours) = 21 / 22.5 = **93.3% complete**

### 1.3 Key Accomplishments

- ✅ Created `blitzy/documentation/scapy_0925ada48540.md` (697 lines) covering all 6 AAP-scoped questions
- ✅ Traced and documented the complete build pipeline: `build()` → `do_build()` → `self_build()` → `post_build()` with cache gate
- ✅ Identified and documented the `_raw_packet_cache_field_value()` blind spot as root cause of the dual-state problem
- ✅ Documented the undocumented `do_build()` cache gate that skips `post_build()` when cache is valid
- ✅ Documented why `copy()` preserves cache (does not fix dual-state) and the correct `clear_cache() + del` pattern
- ✅ Created 4 Mermaid diagrams: build pipeline, cache invalidation comparison, dissection cache population, do_build cache gate
- ✅ Verified all 23 source code line-number citations against actual source files (all correct)
- ✅ Passed 23/23 empirical verification tests confirming every documented behavioral claim
- ✅ Zero source code modifications (documentation-only, per explicit constraint)
- ✅ Clean git working tree with no leftover temporary files

### 1.4 Critical Unresolved Issues

| Issue | Impact | Owner | ETA |
|-------|--------|-------|-----|
| No critical issues | N/A | N/A | N/A |

All AAP-scoped deliverables are fully implemented, verified, and committed. No blocking issues remain.

### 1.5 Access Issues

No access issues identified. The repository is accessible, Python 3.12.3 is available, Scapy imports correctly from the local repository, and all verification scripts executed successfully.

### 1.6 Recommended Next Steps

1. **[High]** Review the document (`blitzy/documentation/scapy_0925ada48540.md`) for technical accuracy and merge the PR
2. **[Low]** If merging is delayed and the Scapy codebase changes, re-verify source code line-number citations
3. **[Low]** Consider integrating the document into Scapy's Sphinx documentation build (currently out of scope per AAP)

## 2. Project Hours Breakdown

### 2.1 Completed Work Detail

| Component | Hours | Description |
|-----------|-------|-------------|
| Source Code Analysis | 4.0 | Deep reading of `scapy/packet.py` (2553 lines), `scapy/fields.py`, `scapy/layers/inet.py`, `scapy/compat.py`, `scapy/base_classes.py` to trace build, dissect, cache, and invalidation code paths |
| Section 1 — Field Calculation Order | 2.5 | Documented complete build pipeline with `post_build()` ordering analysis; traced IP, TCP, UDP implementations; created build pipeline Mermaid flowchart |
| Section 2 — show2() Behavior | 1.5 | Documented `show2()` internals, idempotency proof, and comparison table (show vs show2 vs bytes) |
| Section 3 — Dual-State Problem | 3.0 | Documented reproduction, root cause (`_raw_packet_cache_field_value` blind spot), same-layer vs nested modification, and show/bytes disagreement; created comparison Mermaid diagram |
| Section 4 — Cache Lifecycle | 3.0 | Documented cache population during dissection, cache check during build, all 6 invalidation triggers, and the `do_build()` cache gate; created 2 Mermaid diagrams |
| Section 5 — copy() vs clear_cache() | 2.0 | Documented `copy()` cache preservation, why it doesn't fix dual-state, and the correct `clear_cache() + del` pattern with working example |
| Section 6 — Practical Remedies | 1.0 | Created summary table and universal "safest pattern" code example |
| Empirical Verification | 2.0 | Wrote and executed 23 empirical tests across all 6 sections to confirm every documented behavioral claim |
| Line Reference Verification & Fixes | 1.5 | Verified all source code line citations; applied 2 fix commits correcting 3 line-reference inconsistencies and 1 inaccurate behavioral claim |
| Document Structure & Formatting | 0.5 | Title, preamble, source references table, table of contents, section separators, consistent Markdown formatting |
| **Total Completed** | **21.0** | |

### 2.2 Remaining Work Detail

| Category | Hours | Priority |
|----------|-------|----------|
| PR Review and Merge | 1.0 | High |
| Line Number Freshness Audit (if codebase changes before merge) | 0.5 | Low |
| **Total Remaining** | **1.5** | |

### 2.3 Hours Verification

- Section 2.1 Total (Completed): **21.0 hours**
- Section 2.2 Total (Remaining): **1.5 hours**
- Sum: 21.0 + 1.5 = **22.5 hours** (matches Total Project Hours in Section 1.2 ✅)

## 3. Test Results

All tests were executed by Blitzy's autonomous validation system during the Final Validator phase.

| Test Category | Framework | Total Tests | Passed | Failed | Coverage % | Notes |
|--------------|-----------|-------------|--------|--------|------------|-------|
| Section 1 — Build Pipeline | Python/Scapy | 4 | 4 | 0 | 100% | Verified `raw(x)==bytes(x)`, IP/UDP length/checksum computation in `post_build()` |
| Section 2 — show2() Behavior | Python/Scapy | 3 | 3 | 0 | 100% | Verified idempotency, non-mutation of `self`, show() vs show2() auto-field behavior |
| Section 3 — Dual-State Problem | Python/Scapy | 4 | 4 | 0 | 100% | Confirmed `bytes()` returns original cache, `show()` sees modifications, dual-state existence, same-layer cache invalidation |
| Section 4 — Cache Lifecycle | Python/Scapy | 5 | 5 | 0 | 100% | Confirmed cache population during dissection, `explicit=1`, `setfieldval()` cache clearing, `do_build()` cache gate |
| Section 5 — copy() vs clear_cache() | Python/Scapy | 4 | 4 | 0 | 100% | Confirmed `copy()` preserves cache (does NOT fix dual-state); `clear_cache()` DOES fix dual-state |
| Section 6 — Practical Remedies | Python/Scapy | 3 | 3 | 0 | 100% | Confirmed `clear_cache()` alone leaves stale checksum, `clear_cache()+del` correctly recomputes |
| **Total** | | **23** | **23** | **0** | **100%** | All claims in documentation empirically validated |

Additionally, 6 independent core verification tests were re-executed during project assessment, all passing:
1. ✅ Cache populated after dissection
2. ✅ Same-layer modification correctly rebuilds
3. ✅ Dual-state confirmed (bytes returns original despite payload modification)
4. ✅ copy() preserves cache (does not fix dual-state)
5. ✅ clear_cache() fixes dual-state
6. ✅ show2() is idempotent

## 4. Runtime Validation & UI Verification

### Runtime Health

- ✅ **Scapy Library Import**: Scapy imports correctly from local repository (`scapy.VERSION = 2026.04.09`)
- ✅ **Python Runtime**: Python 3.12.3 compatible (project requires ≥3.7, <4)
- ✅ **Packet Construction**: `IP() / UDP() / Raw()` builds correctly
- ✅ **Packet Dissection**: Wire bytes dissect correctly with cache population
- ✅ **Cache Mechanism**: `raw_packet_cache` populates on dissection and clears on `setfieldval()`
- ✅ **Dual-State Reproduction**: The documented dual-state behavior is reproducible and matches documentation
- ✅ **Fix Verification**: `clear_cache() + del` pattern correctly resolves dual-state

### Document Verification

- ✅ **File Integrity**: `blitzy/documentation/scapy_0925ada48540.md` — 697 lines, well-formed Markdown
- ✅ **Mermaid Diagrams**: 4 diagrams found, all with valid syntax (`flowchart TD` / `flowchart LR`)
- ✅ **Code Blocks**: 29 fenced code blocks (25 Python, 4 Mermaid)
- ✅ **Tables**: 10 Markdown tables with correct alignment
- ✅ **Section Structure**: 6 major sections, 19 subsections, matching AAP §0.4.1 outline
- ✅ **Source Citations**: All 23 line-number citations verified against actual source files
- ✅ **Git Status**: Clean working tree, no uncommitted changes

### UI/API Verification

Not applicable — this is a documentation-only project with no UI or API components.

## 5. Compliance & Quality Review

| Compliance Benchmark | Status | Details |
|---------------------|--------|---------|
| **AAP Req 1 — Field Calculation Order** | ✅ Pass | Section 1 (1.1–1.4): Complete build pipeline trace, `post_build()` ordering, IP/TCP/UDP examples, Mermaid diagram |
| **AAP Req 2 — show2() Behavior** | ✅ Pass | Section 2 (2.1–2.3): Implementation analysis, idempotency proof, comparison table |
| **AAP Req 3 — Dual-State Problem** | ✅ Pass | Section 3 (3.1–3.4): Reproduction code, root cause trace (`_raw_packet_cache_field_value`), comparison diagram |
| **AAP Req 4 — Cache Invalidation** | ✅ Pass | Section 4 (4.1–4.4): Complete lifecycle, all 6 invalidation triggers, `do_build()` cache gate |
| **AAP Req 5 — Same-Layer vs Nested** | ✅ Pass | Section 3.3: Detailed comparison of `__setattr__` code paths, Mermaid comparison diagram |
| **AAP Req 6 — copy() vs clear_cache()** | ✅ Pass | Section 5 (5.1–5.3): `copy()` cache preservation proof, `clear_cache() + del` pattern with working example |
| **Inferred: do_build() cache gate** | ✅ Pass | Section 4.4: Documented the undocumented `post_build()` skip behavior |
| **Inferred: Dissected explicit fields** | ✅ Pass | Section 5.3: Documented why `del pkt.chksum` is required after `clear_cache()` |
| **Inferred: copy() preserves cache** | ✅ Pass | Section 5.1: Line-by-line trace of `copy()` at L407-426 |
| **No Source Code Modifications** | ✅ Pass | `git diff` confirms zero changes to `scapy/`, `doc/`, `test/` directories |
| **No Leftover Temp Files** | ✅ Pass | `git status` clean, no temporary test scripts remain |
| **Source Code Citations** | ✅ Pass | All 23 line-number references verified against actual files |
| **Empirical Verification** | ✅ Pass | 23/23 tests passed confirming all documented claims |
| **Mermaid Diagram Syntax** | ✅ Pass | 4 diagrams validated with correct `flowchart` syntax |
| **Document Structure** | ✅ Pass | Matches AAP §0.4.1 outline exactly (6 sections, 19 subsections) |

### Fixes Applied During Validation

| Fix | Commit | Description |
|-----|--------|-------------|
| Line Reference Fix | `63e976e3` | Corrected 3 line-reference inconsistencies in cache internals documentation |
| Behavioral Claim Fix | `157cc132` | Corrected inaccurate behavioral claim in Section 5.3 code example |

## 6. Risk Assessment

| Risk | Category | Severity | Probability | Mitigation | Status |
|------|----------|----------|-------------|------------|--------|
| Line numbers become stale if Scapy codebase changes | Technical | Low | Medium | Re-verify line citations before merge if significant time passes; line numbers are pinned to the current commit | Monitored |
| Document not integrated into Sphinx docs | Operational | Low | N/A | Explicitly out of scope per AAP; standalone Markdown file readable on GitHub/GitLab | Accepted |
| Missing Sphinx autodoc for `scapy/packet.py` internals | Technical | Low | Low | The deep-dive document fills this gap for the specific mechanisms covered; broader autodoc is a separate initiative | Accepted |
| Mermaid rendering depends on viewer support | Technical | Low | Low | GitHub, GitLab, and most modern Markdown viewers support Mermaid natively; fallback is plain text node labels | Accepted |

No high-severity or high-probability risks were identified. This is a documentation-only change with no runtime impact on the Scapy library.

## 7. Visual Project Status

```mermaid
pie title Project Hours Breakdown
    "Completed Work" : 21
    "Remaining Work" : 1.5
```

**Breakdown of Completed Work by Section:**

| Section | Hours | % of Total |
|---------|-------|------------|
| Source Code Analysis | 4.0 | 17.8% |
| Section 1 — Field Calculation Order | 2.5 | 11.1% |
| Section 2 — show2() Behavior | 1.5 | 6.7% |
| Section 3 — Dual-State Problem | 3.0 | 13.3% |
| Section 4 — Cache Lifecycle | 3.0 | 13.3% |
| Section 5 — copy() vs clear_cache() | 2.0 | 8.9% |
| Section 6 — Practical Remedies | 1.0 | 4.4% |
| Empirical Verification | 2.0 | 8.9% |
| Line Ref Verification & Fixes | 1.5 | 6.7% |
| Document Structure & Formatting | 0.5 | 2.2% |

**Remaining Work:**

| Category | Hours |
|----------|-------|
| PR Review and Merge | 1.0 |
| Line Number Freshness Audit | 0.5 |
| **Total Remaining** | **1.5** |

## 8. Summary & Recommendations

### Achievement Summary

The project successfully delivered a comprehensive 697-line technical deep-dive document (`blitzy/documentation/scapy_0925ada48540.md`) that answers all six questions specified in the Agent Action Plan. The document traces Scapy's build pipeline, cache lifecycle, and the root cause of the "dual-state" problem directly from source code, with every claim backed by specific line-number citations and validated by 23 empirical tests (all passing).

The project is **93.3% complete** (21 hours completed out of 22.5 total hours). All AAP-scoped autonomous work is finished. The remaining 1.5 hours consist entirely of human review activities (PR review/merge and optional line-number freshness audit).

### Key Findings Documented

1. The `do_build()` cache gate at `scapy/packet.py:737-740` skips `post_build()` entirely when `raw_packet_cache` is valid — this is the root cause of stale checksums/lengths, not a miscalculation within `post_build()`.
2. `_raw_packet_cache_field_value()` at L648 only tracks `x.fields` for `PacketListField` sub-packets, creating a blind spot for payload modifications.
3. `copy()` at L416 explicitly preserves `raw_packet_cache`, so it does NOT fix the dual-state problem.
4. The correct fix is `clear_cache() + del pkt[Layer].chksum/len` to both reset the cache and restore auto-compute defaults.

### Remaining Gaps

- **PR Review**: The document needs human review for technical accuracy before merging.
- **Sphinx Integration**: The document is standalone Markdown, not integrated into Scapy's Sphinx documentation build (explicitly out of scope per AAP).

### Production Readiness Assessment

The deliverable is **production-ready for merge**. The document is complete, well-structured, empirically validated, and committed to the correct branch with a clean working tree. No blocking issues remain.

### Success Metrics

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| Questions Answered | 6 | 6 | ✅ Met |
| Source Line Citations Verified | 100% | 100% (23/23) | ✅ Met |
| Empirical Tests Passing | 100% | 100% (23/23) | ✅ Met |
| Mermaid Diagrams | ≥4 | 4 | ✅ Met |
| Source Files Modified | 0 | 0 | ✅ Met |
| Temporary Files Remaining | 0 | 0 | ✅ Met |

## 9. Development Guide

### System Prerequisites

| Requirement | Version | Purpose |
|-------------|---------|---------|
| Python | ≥ 3.7, < 4 (3.12.3 tested) | Runtime for Scapy |
| Git | Any recent version | Repository management |
| Markdown viewer | GitHub, VS Code, or `grip` | Viewing the documentation |

No additional packages are required. Scapy has zero mandatory external dependencies on Linux.

### Environment Setup

```bash
# Clone the repository
git clone https://github.com/blitzy-research/scapy.git
cd scapy

# Checkout the feature branch
git checkout blitzy-0fe0bb58-e2c1-4f97-9845-605c3f83b4c9

# (Optional) Create a virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install Scapy from local repository (editable mode)
pip install -e .
```

### Dependency Installation

```bash
# Scapy has no mandatory external dependencies on Linux
# Verify installation:
python3 -c "import scapy; print('Scapy version:', scapy.VERSION)"
# Expected output: Scapy version: 2026.04.09
```

### Viewing the Documentation

```bash
# Option 1: View on GitHub (Mermaid diagrams render natively)
# Navigate to: blitzy/documentation/scapy_0925ada48540.md

# Option 2: View locally with VS Code
code blitzy/documentation/scapy_0925ada48540.md

# Option 3: View in terminal
cat blitzy/documentation/scapy_0925ada48540.md

# Option 4: View with grip (GitHub-flavored Markdown preview)
pip install grip
grip blitzy/documentation/scapy_0925ada48540.md
# Opens browser at http://localhost:6419
```

### Verification Steps

```bash
# 1. Verify the document exists and has expected line count
wc -l blitzy/documentation/scapy_0925ada48540.md
# Expected: 697 blitzy/documentation/scapy_0925ada48540.md

# 2. Verify no source files were modified
git diff --name-only origin/scapy_0925ada48540...HEAD -- scapy/ doc/ test/
# Expected: (no output — zero modified source files)

# 3. Verify the document's key claims empirically
python3 -c "
from scapy.packet import Packet, Raw
from scapy.fields import ByteField, FieldLenField, PacketListField

class Inner(Packet):
    fields_desc = [ByteField('val', 0)]

class Outer(Packet):
    fields_desc = [
        FieldLenField('count', None, count_of='items'),
        PacketListField('items', [], Inner, count_from=lambda p: p.count)
    ]

# Build, dissect, modify payload, verify dual-state
orig = bytes(Outer(items=[Inner(val=1)]))
pkt = Outer(orig)
pkt.items[0].payload = Raw(b'injected')
assert bytes(pkt) == orig, 'Dual-state NOT confirmed'
print('Dual-state confirmed: bytes() returns original despite payload mod')

# Verify fix
pkt.clear_cache()
assert bytes(pkt) != orig, 'Fix did NOT work'
print('Fix confirmed: clear_cache() resolves dual-state')
"
# Expected:
# Dual-state confirmed: bytes() returns original despite payload mod
# Fix confirmed: clear_cache() resolves dual-state

# 4. Verify source code line citations
python3 -c "
with open('scapy/packet.py') as f:
    lines = f.readlines()
checks = {592:'__bytes__', 678:'self_build', 724:'do_build', 746:'build',
           758:'post_build', 648:'_raw_packet_cache_field_value', 664:'clear_cache'}
for n, exp in checks.items():
    assert exp in lines[n-1], f'L{n}: expected {exp}'
    print(f'OK L{n}: {exp}')
"
# Expected: OK for all 7 line references
```

### Troubleshooting

| Issue | Cause | Resolution |
|-------|-------|------------|
| `ModuleNotFoundError: No module named 'scapy'` | Scapy not installed | Run `pip install -e .` from repo root |
| Mermaid diagrams not rendering | Viewer doesn't support Mermaid | Use GitHub, VS Code with Mermaid extension, or Mermaid Live Editor |
| Line number citations don't match | Codebase changed since documentation was written | Re-verify citations with `python3 -c` script above; update document if needed |
| `PermissionError` when importing Scapy | Raw socket permissions | Not needed for documentation verification; use `--no-privileges` or run verification as above |

## 10. Appendices

### A. Command Reference

| Command | Purpose |
|---------|---------|
| `git checkout blitzy-0fe0bb58-e2c1-4f97-9845-605c3f83b4c9` | Switch to the feature branch |
| `pip install -e .` | Install Scapy from local repository |
| `python3 -c "import scapy; print(scapy.VERSION)"` | Verify Scapy installation |
| `wc -l blitzy/documentation/scapy_0925ada48540.md` | Verify document line count |
| `git diff --name-only origin/scapy_0925ada48540...HEAD` | Verify only documentation was changed |
| `git log --oneline origin/scapy_0925ada48540...HEAD` | View commits on feature branch |

### C. Key File Locations

| File | Purpose |
|------|---------|
| `blitzy/documentation/scapy_0925ada48540.md` | **Deliverable** — The comprehensive cache internals documentation |
| `scapy/packet.py` | Primary source — Core `Packet` class (build, dissect, cache, copy, show) |
| `scapy/fields.py` | Supporting source — Field types, `PacketListField`, cache tracking |
| `scapy/layers/inet.py` | Reference source — IP, TCP, UDP `post_build()` implementations |
| `scapy/compat.py` | Supporting source — `raw()` function definition |
| `scapy/base_classes.py` | Supporting source — `Packet_metaclass`, `__all_slots__` |
| `doc/scapy/build_dissect.rst` | Existing docs — Build/dissect pipeline (cross-referenced in deliverable) |

### D. Technology Versions

| Technology | Version | Notes |
|------------|---------|-------|
| Python | 3.12.3 (tested), ≥3.7 required | Per `pyproject.toml` `requires-python` |
| Scapy | 2026.04.09 (local build) | Installed from repository |
| pip | 25.3 | Package manager |
| Git | Available | Repository management |
| Markdown / Mermaid | N/A | Document format; rendered by GitHub/VS Code |

### G. Glossary

| Term | Definition |
|------|-----------|
| `raw_packet_cache` | Byte string stored on a `Packet` instance after dissection, containing the exact wire bytes consumed by that layer. Used by `self_build()` to skip field-by-field serialization when the cache is still valid. |
| `raw_packet_cache_fields` | Dictionary stored alongside `raw_packet_cache`, tracking deep-copied snapshots of mutable field values at dissection time. Used by `self_build()` to detect whether mutable fields have changed since dissection. |
| `post_build()` | Method called by `do_build()` after `self_build()` and `do_build_payload()`. Computes derived fields (lengths, checksums). Skipped by the cache gate when `raw_packet_cache` is not `None`. |
| Cache Gate | The conditional at `do_build()` L737-740 that decides whether to call `post_build()` (cache invalid) or return `pkt + pay` directly (cache valid). |
| Dual-State Problem | The condition where a packet's in-memory object graph reflects a modification (visible via `show()`) but its serialized form returns original cached bytes (via `bytes()`), because the cache comparison fails to detect the change. |
| `PacketListField` | A field type that holds a list of `Packet` instances. Its cache tracking only monitors `x.fields` for each sub-packet, not their payloads. |
| `clear_cache()` | Method that recursively sets `raw_packet_cache = None` on the packet, all held sub-packets, and the entire payload chain. |
| `__all_slots__` | Set built by `Packet_metaclass` containing all `__slots__` from the MRO. Used by `__setattr__` to distinguish Python slot attributes (like `payload`) from protocol fields. |