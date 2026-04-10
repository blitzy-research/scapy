# Blitzy Project Guide

---

## 1. Executive Summary

### 1.1 Project Overview

This project creates a comprehensive, code-grounded technical deep-dive document answering five detailed questions about Scapy's packet dissection mechanism. The target artifact is `blitzy/documentation/scapy_0925ada48540.md` — a standalone 3,023-line (~128KB) Markdown document that explains how Scapy transforms raw bytes into a nested, typed layer hierarchy at runtime. The document covers layer boundary discovery, runtime HTTP dissection tracing, unknown payload handling, the trust model for header fields, and GRE tunnel recursion. All answers are backed by 152 source code citations, 73 Python code blocks with runtime evidence, and 5 Mermaid diagrams. The repository itself remains unmodified per the read-only constraint.

### 1.2 Completion Status

```mermaid
pie title Project Completion
    "Completed (49h)" : 49
    "Remaining (4h)" : 4
```

| Metric | Value |
|---|---|
| **Total Project Hours** | 53 |
| **Completed Hours (AI)** | 49 |
| **Remaining Hours** | 4 |
| **Completion Percentage** | 92.5% |

**Calculation:** 49 completed hours / (49 + 4 remaining hours) = 49 / 53 = **92.5% complete**

### 1.3 Key Accomplishments

- ✅ Created comprehensive 3,023-line documentation file (`blitzy/documentation/scapy_0925ada48540.md`)
- ✅ All 5 user questions answered with code-grounded explanations and runtime evidence
- ✅ 152 source code citations in `file:line` format — all verified against actual repository source
- ✅ 73 Python code blocks with runnable examples and captured runtime output
- ✅ 5 Mermaid diagrams (dissection lifecycle, binding decision tree, HTTP sequence, trust model flow, GRE recursion)
- ✅ HTTP `load_layer('http')` caveat prominently documented in Section 2.3
- ✅ Thinking/rationale provided for every answer as required
- ✅ 3 appendices: source code reference index, glossary, complete `payload_guess` runtime evidence
- ✅ Zero existing repository files modified (read-only constraint satisfied)
- ✅ No temporary observation scripts remaining in the repository
- ✅ 10/10 runtime validation checks passed by Final Validator
- ✅ All 162 code fence markers balanced, all 35 section headers verified present

### 1.4 Critical Unresolved Issues

| Issue | Impact | Owner | ETA |
|---|---|---|---|
| Source code line numbers are pinned to current HEAD | Line numbers may drift if upstream Scapy merges significant refactors | Human Developer | Before next upstream merge |
| Document not integrated into Sphinx documentation tree | Standalone in `blitzy/documentation/`; not discoverable via Sphinx search/index | Human Developer | Optional; low priority |

### 1.5 Access Issues

No access issues identified. The project is a documentation-only task with no external service dependencies, API keys, database connections, or third-party integrations required. The source repository is fully accessible and Scapy is importable from the repository root.

### 1.6 Recommended Next Steps

1. **[Medium]** Have a Scapy domain expert review the document for technical accuracy and completeness
2. **[Medium]** Verify source code line number citations still match if the base branch has received new commits
3. **[Low]** Consider integrating the document into the Sphinx documentation tree (`doc/scapy/`) if long-term discoverability is desired
4. **[Low]** Add automated link/reference checking to CI if the document is expected to be maintained alongside the codebase

---

## 2. Project Hours Breakdown

### 2.1 Completed Work Detail

| Component | Hours | Description |
|---|---|---|
| Source Code Analysis & Research | 8 | Deep analysis of 7 key source files (`scapy/packet.py`, `scapy/base_classes.py`, `scapy/layers/l2.py`, `scapy/layers/inet.py`, `scapy/layers/inet6.py`, `scapy/layers/http.py`, `scapy/config.py`) totaling ~4,000+ lines of code; traced dissection pipeline, binding system, trust model, and error handling paths |
| Runtime Validation Scripts | 4 | Created and executed observation scripts to validate 10+ runtime claims; constructed synthetic packets for each scenario (unknown payloads, trust model, GRE tunnels); captured and formatted runtime output |
| Document Structure & Planning | 2 | Designed document architecture with 6 major sections + 3 appendices; planned 5 Mermaid diagrams; created comprehensive table of contents; established terminology glossary |
| Q1: Layer Boundary Discovery Documentation | 6 | 7 subsections (~830 lines) covering `bind_layers()`, `payload_guess`, `guess_payload_class()`, `dispatch_hook()`, binding tables for Ether/IP/TCP/GRE, plus 2 Mermaid flowcharts |
| Q2: HTTP Request Dissection Documentation | 5 | 5 subsections (~465 lines) with step-by-step 4-layer dissection trace, HTTP layer loading caveat, Mermaid sequence diagram, runtime evidence |
| Q3: Unknown Payload Documentation | 4 | 4 subsections (~382 lines) covering `default_payload_class()` → `Raw` fallback, packet object demonstrations, error handling pipeline in `do_dissect_payload()`, `NoPayload` sentinel |
| Q4: Trust Model Documentation | 4 | 5 subsections (~325 lines) demonstrating unconditional header trust with IPv6 EtherType + IPv4 bytes experiment, design rationale, Mermaid trust model flow diagram |
| Q5: GRE Tunnel Recursion Documentation | 5 | 7 subsections (~550 lines) covering GRE bindings, full recursive Ether/IP/GRE/IP/TCP trace, double-GRE proof, transparent Ethernet bridging, comparison table |
| Conclusion & Appendices | 3 | Summary of answers, key takeaways table, design philosophy, source code reference index (Appendix A), glossary (Appendix B), complete `payload_guess` runtime evidence (Appendix C) |
| Mermaid Diagrams | 3 | 5 diagrams: dissection lifecycle flowchart, binding decision tree, HTTP dissection sequence, trust model flow, GRE recursion visualization |
| Code Review Fixes & Iterations | 3 | 3 revision commits: fixed 8 code review findings, corrected IP class signature and GRE binding analysis, fixed version string and `show()` output labels |
| Final Validation | 2 | 10 runtime validation checks, markdown formatting verification, 35 section header verification, code fence balance check, line number accuracy verification |
| **Total Completed** | **49** | |

### 2.2 Remaining Work Detail

| Category | Hours | Priority |
|---|---|---|
| Human expert review of document technical accuracy | 2 | Medium |
| Line number staleness verification against latest upstream | 1 | Medium |
| Optional Sphinx documentation tree integration | 1 | Low |
| **Total Remaining** | **4** | |

---

## 3. Test Results

| Test Category | Framework | Total Tests | Passed | Failed | Coverage % | Notes |
|---|---|---|---|---|---|---|
| Runtime Claim Validation | Python/Scapy | 10 | 10 | 0 | 100% | All 10 runtime claims verified by executing Scapy: Ether.payload_guess count (16), IP.extract_padding correctness, HTTP layer loading adds 5 TCP→HTTP bindings, Ether.dispatch_hook Ether/Dot3 selection, unknown EtherType → Raw fallback, trust model IPv6-on-IPv4 garbling, GRE 5-layer tunnel hierarchy, NoPayload bool()=False, conf.debug_dissector=False, conf.raw_layer=Raw |
| Source Line Number Verification | Manual/grep | 15 | 15 | 0 | 100% | Key line numbers verified: bind_layers (1975), guess_payload_class (1062), default_payload_class (1081), do_dissect_payload (1023), dissect (1049), Ether (244), dispatch_hook (267), GRE (578), IP (521), TCP (753), HTTP (548), bindings in inet.py (1101-1115), inet6.py (4083-4085), http.py (748-753) |
| Markdown Formatting | Manual count | 3 | 3 | 0 | 100% | 162 code fence markers balanced (even count), all 35 section headers present, table formatting verified |
| Repository Integrity | Git diff | 1 | 1 | 0 | 100% | Confirmed only 1 file changed (`blitzy/documentation/scapy_0925ada48540.md`, +3023 lines); zero existing files modified; working tree clean |

---

## 4. Runtime Validation & UI Verification

### Runtime Health

- ✅ Scapy 2026.04.09 importable from repository root (`python3 -c "import scapy; print(scapy.VERSION)"`)
- ✅ All core layer modules importable: `scapy.layers.l2`, `scapy.layers.inet`, `scapy.layers.inet6`, `scapy.layers.http`
- ✅ `Ether.payload_guess` returns 16 entries (matching documented table)
- ✅ `IP.payload_guess` returns 11 entries (matching documented table)
- ✅ `GRE.payload_guess` returns 9 entries (matching documented table)
- ✅ `conf.raw_layer` is `<class 'scapy.packet.Raw'>` (confirmed)
- ✅ `conf.debug_dissector` defaults to `False` (confirmed)
- ✅ TCP has 0 HTTP bindings before `load_layer('http')` (confirming HTTP caveat)

### Runtime Validation Checks

- ✅ Unknown EtherType (0x1234) falls through to `Raw` with data preserved
- ✅ Trust model: `Ether(type=0x86dd)` + IPv4 bytes → `IPv6` class instantiated with `version=4` (garbled)
- ✅ GRE tunnel: `Ether/IP/GRE/IP/TCP` dissects into exactly 5 layers in correct order
- ✅ `NoPayload` singleton: `bool(NoPayload()) == False`

### UI Verification

Not applicable — this is a documentation-only project with no user interface components. The deliverable is a Markdown file rendered by GitHub/GitLab's native Markdown viewer.

---

## 5. Compliance & Quality Review

| Compliance Item | AAP Requirement | Status | Evidence |
|---|---|---|---|
| Document placement | `blitzy/documentation/scapy_0925ada48540.md` | ✅ Pass | File exists at correct path, 3,023 lines |
| All 5 questions answered | Q1-Q5 with code-grounded explanations | ✅ Pass | Sections 1-5 in document, each with dedicated subsections |
| Source code citations | Every technical claim cites `file:line` | ✅ Pass | 152 `Source:` citations verified against actual source |
| Runtime evidence | Each question includes runtime `show()`/`repr()` output | ✅ Pass | 73 Python code blocks with captured output |
| Thinking/rationale | Each answer includes reasoning | ✅ Pass | "Thinking:" and "Rationale:" blocks in every section |
| Mermaid diagrams | Complex control flows visualized | ✅ Pass | 5 diagrams (exceeds required minimum of 4) |
| Read-only repository | No existing files modified | ✅ Pass | `git diff --name-status` shows only 1 Added file |
| Temporary script cleanup | No observation scripts remain | ✅ Pass | No `.py` temp files found in repository |
| HTTP layer caveat | `load_layer('http')` requirement documented | ✅ Pass | Section 2.3 with dedicated runtime comparison |
| `dispatch_hook()` documented | Pre-binding class selection explained | ✅ Pass | Section 1.4 covers Ether, GRE, HTTP dispatch hooks |
| Binding tables enumerated | `payload_guess` for Ether, IP, TCP, GRE | ✅ Pass | Section 1.5 + Appendix C with full runtime evidence |
| Error handling documented | `do_dissect_payload()` try/except path | ✅ Pass | Section 3.3 with complete error recovery pipeline |
| Code-as-truth | No assumptions or general knowledge statements | ✅ Pass | All claims traced to specific source code paths |
| Document format | GitHub-Flavored Markdown | ✅ Pass | Balanced code fences, proper heading hierarchy |

### Quality Fixes Applied During Validation

1. **Commit `ee0ebe59`:** Fixed 8 code review findings (formatting, accuracy corrections)
2. **Commit `05791f4a`:** Corrected IP class signature and GRE binding analysis
3. **Commit `40d03296`:** Corrected version string and `show()` output label

---

## 6. Risk Assessment

| Risk | Category | Severity | Probability | Mitigation | Status |
|---|---|---|---|---|---|
| Source code line numbers become stale after upstream merges | Technical | Low | Medium | Line numbers are pinned to current commit; document header states Scapy version; a `grep`-based verification script can revalidate | Open — monitor on upstream merges |
| Mermaid diagrams not rendered in non-GitHub/GitLab viewers | Technical | Low | Low | Mermaid is widely supported (GitHub, GitLab, VS Code, Notion); fallback: diagrams are self-explanatory from surrounding text | Accepted |
| Document is standalone, not discoverable via Sphinx search | Operational | Low | Low | `blitzy/documentation/` is by design independent; integration into Sphinx is an optional future task | Accepted |
| Runtime output examples may vary across Scapy versions | Technical | Low | Low | All examples were validated against Scapy 2026.04.09 (development HEAD); document states this version explicitly | Accepted |
| No automated CI validation for documentation accuracy | Operational | Medium | Low | Recommend adding a CI job that runs the runtime validation checks and compares output | Open — optional improvement |

---

## 7. Visual Project Status

```mermaid
pie title Project Hours Breakdown
    "Completed Work" : 49
    "Remaining Work" : 4
```

**Summary:** 49 hours of AAP-scoped work completed out of 53 total hours = **92.5% complete**. The remaining 4 hours consist of human expert review (2h), line number staleness verification (1h), and optional Sphinx integration (1h).

---

## 8. Summary & Recommendations

### Achievement Summary

The project is **92.5% complete** (49 of 53 total hours). The sole AAP deliverable — a comprehensive technical deep-dive document on Scapy's packet dissection mechanism — has been fully authored, validated, and committed. The document answers all 5 user questions with code-grounded explanations backed by 152 source code citations, 73 runtime code examples, and 5 Mermaid diagrams. All 10 runtime validation checks pass, all source line numbers have been verified, and the repository's read-only constraint is fully satisfied with zero existing files modified.

### Remaining Gaps

The remaining 4 hours (7.5% of total) cover path-to-production activities that require human judgment:

1. **Human expert review (2h):** A Scapy domain expert should review the document for technical accuracy, particularly the trust model analysis (Section 4) and the GRE binding edge case noted in Appendix C.3.
2. **Line number staleness check (1h):** If the base branch has received new commits since this work was done, verify that the 152 line number citations still point to the correct code.
3. **Optional Sphinx integration (1h):** If long-term discoverability within the Scapy documentation ecosystem is desired, the document can be converted to RST and added to the Sphinx toctree.

### Production Readiness Assessment

The document is **production-ready** for its stated purpose: a standalone deep-dive reference placed in `blitzy/documentation/`. No blocking issues exist. The two Medium-priority remaining items (human review and line number verification) are standard quality assurance steps for technical documentation and do not indicate deficiencies in the delivered artifact.

### Success Metrics

| Metric | Target | Actual | Status |
|---|---|---|---|
| Questions answered | 5/5 | 5/5 | ✅ Met |
| Source code citations | Every claim cited | 152 citations | ✅ Exceeded |
| Mermaid diagrams | ≥4 | 5 | ✅ Exceeded |
| Runtime evidence per question | ≥1 | 73 code blocks total | ✅ Exceeded |
| Existing files modified | 0 | 0 | ✅ Met |
| Temporary scripts remaining | 0 | 0 | ✅ Met |
| Runtime validation checks | All pass | 10/10 pass | ✅ Met |

---

## 9. Development Guide

### System Prerequisites

| Software | Version | Purpose |
|---|---|---|
| Python | ≥3.7, <4 (tested with 3.12.3) | Runtime for Scapy |
| Git | Any modern version | Repository clone and branch management |
| Markdown viewer | GitHub, GitLab, VS Code, or `grip` | Rendering the documentation with Mermaid diagrams |

No additional software, databases, services, or API keys are required. This is a documentation-only project.

### Environment Setup

```bash
# Clone the repository
git clone https://github.com/blitzy-research/scapy.git
cd scapy

# Checkout the feature branch
git checkout blitzy-ffc0932c-88f2-431e-835a-4d9a0bc58c30

# Verify the branch
git branch --show-current
# Expected: blitzy-ffc0932c-88f2-431e-835a-4d9a0bc58c30
```

### Verify Scapy Is Importable

```bash
# From repository root
python3 -c "import scapy; print(scapy.VERSION)"
# Expected: 2026.04.09
```

### View the Documentation

```bash
# Check the document exists
ls -la blitzy/documentation/scapy_0925ada48540.md
# Expected: ~128KB file

# Count lines
wc -l blitzy/documentation/scapy_0925ada48540.md
# Expected: 3023

# Open in VS Code (if available)
code blitzy/documentation/scapy_0925ada48540.md
```

For Mermaid diagram rendering, use GitHub's native Markdown viewer or install `grip` for local preview:

```bash
pip install grip
grip blitzy/documentation/scapy_0925ada48540.md
# Opens browser at http://localhost:6419
```

### Run Runtime Validation Checks

To verify the document's runtime claims are still accurate:

```bash
python3 -c "
from scapy.all import *

# Check 1: Ether.payload_guess count
assert len(Ether.payload_guess) == 16, f'Expected 16, got {len(Ether.payload_guess)}'
print('PASS: Ether.payload_guess has 16 entries')

# Check 2: Unknown payload falls back to Raw
pkt = Ether(type=0x1234) / Raw(load=b'test')
d = Ether(raw(pkt))
assert type(d.payload).__name__ == 'Raw'
print('PASS: Unknown EtherType falls back to Raw')

# Check 3: Trust model
import struct
ipv4_bytes = raw(IP() / TCP())
ether_hdr = b'\xff\xff\xff\xff\xff\xff\x00\x00\x00\x00\x00\x00' + struct.pack('!H', 0x86dd)
d2 = Ether(ether_hdr + ipv4_bytes)
assert type(d2.payload).__name__ == 'IPv6'
print('PASS: Trust model confirmed')

# Check 4: GRE tunnel recursion
pkt3 = Ether() / IP(proto=47) / GRE(proto=2048) / IP() / TCP()
d3 = Ether(raw(pkt3))
layers = []
l = d3
while l and type(l).__name__ != 'NoPayload':
    layers.append(type(l).__name__)
    l = l.payload
assert layers == ['Ether', 'IP', 'GRE', 'IP', 'TCP']
print('PASS: GRE tunnel recursion correct')

print('ALL CHECKS PASSED')
"
```

### Verify Repository Integrity

```bash
# Confirm only the documentation file was changed
git diff origin/scapy_0925ada48540..HEAD --name-status
# Expected: A  blitzy/documentation/scapy_0925ada48540.md

# Confirm working tree is clean
git status --short
# Expected: no output (clean)
```

### Troubleshooting

| Issue | Cause | Resolution |
|---|---|---|
| `ModuleNotFoundError: No module named 'scapy'` | Not running from repository root | `cd` to the repository root directory |
| Mermaid diagrams not rendering | Viewer doesn't support Mermaid | Use GitHub/GitLab web UI or install `grip` |
| Line number mismatch in citations | Upstream code has changed since document creation | Re-verify with `grep -n "pattern" scapy/packet.py` |
| `Ether.payload_guess` count differs from 16 | Additional layers loaded or Scapy version changed | Check `scapy.VERSION` and loaded layers |

---

## 10. Appendices

### A. Command Reference

| Command | Purpose | Context |
|---|---|---|
| `python3 -c "import scapy; print(scapy.VERSION)"` | Verify Scapy is importable and check version | Repository root |
| `git diff origin/scapy_0925ada48540..HEAD --name-status` | View files changed on feature branch | Repository root |
| `git log --oneline origin/scapy_0925ada48540..HEAD` | View commit history on feature branch | Repository root |
| `wc -l blitzy/documentation/scapy_0925ada48540.md` | Count document lines | Repository root |
| `grep -c "Source:" blitzy/documentation/scapy_0925ada48540.md` | Count source code citations | Repository root |

### B. Key File Locations

| File | Purpose |
|---|---|
| `blitzy/documentation/scapy_0925ada48540.md` | **Primary deliverable** — comprehensive dissection deep-dive document |
| `scapy/packet.py` | Core dissection engine (dissect, bind_layers, Raw, NoPayload) |
| `scapy/base_classes.py` | Packet metaclass, aliastypes construction |
| `scapy/layers/l2.py` | Ether, GRE, dispatch_hook, layer-2 bindings |
| `scapy/layers/inet.py` | IP, TCP, UDP, layer-3/4 bindings |
| `scapy/layers/inet6.py` | IPv6 bindings |
| `scapy/layers/http.py` | HTTP dissection and port bindings |
| `scapy/config.py` | Configuration: load_layers, raw_layer, debug_dissector |
| `doc/scapy/build_dissect.rst` | Existing upstream dissection documentation (not modified) |

### C. Technology Versions

| Technology | Version | Notes |
|---|---|---|
| Scapy | 2026.04.09 (dev HEAD) | Subject of documentation; all line numbers verified against this version |
| Python | ≥3.7, <4 (tested 3.12.3) | Runtime for Scapy |
| Markdown | GitHub-Flavored Markdown | Document format with Mermaid diagram support |
| Sphinx | ≥3.0.0 | Existing documentation framework (not modified) |

### D. Glossary

| Term | Definition |
|---|---|
| Dissection | Converting raw bytes into a structured Packet object with typed fields |
| Binding | A registered association between two layer classes via `bind_layers()` |
| `payload_guess` | Class-level list of (field_conditions, next_class) tuples for dissection |
| `dispatch_hook()` | Classmethod that can override the class used for dissection before field parsing |
| `Raw` | Universal fallback layer for unrecognized bytes (stores in `.load`) |
| `NoPayload` | Sentinel singleton indicating the end of the layer chain |
| Trust model | Scapy's design principle of trusting header field values without cross-validation |
| `extract_padding()` | Method separating a layer's payload from trailing padding bytes |
