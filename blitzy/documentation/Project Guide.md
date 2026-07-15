# Blitzy Project Guide
### Scapy Runtime Investigation — Build → Send → Sniff Answer Document

> **Brand color legend (applied throughout):** Completed / AI Work = **Dark Blue `#5B39F3`** · Remaining / Not Completed = **White `#FFFFFF`** · Headings / Accents = **Violet-Black `#B23AF2`** · Highlight = **Mint `#A8FDD9`**.

---

## 1. Executive Summary

### 1.1 Project Overview

This project answers a runtime question about the **Scapy** packet-manipulation library **by observation, not by reading alone**: when a user starts Scapy in its default configuration, builds a simple multi-layer packet (`Ether()/IP()/TCP()`), sends it, and sniffs for that same traffic, **what does Scapy actually display at each stage, and why?** The audience is engineers and reviewers who need a precise, reproducible account of Scapy's build/send/sniff behavior. The deliverable is a single evidence-grounded Markdown document in which every behavioral claim is paired with the exact observed output and a `file:line` citation into the source. This is a read-only investigation: the sole committed artifact is the answer document; no Scapy source, test, configuration, or build file is changed.

### 1.2 Completion Status

```mermaid
%%{init: {"theme": "base", "themeVariables": {"pie1": "#5B39F3", "pie2": "#FFFFFF", "pieStrokeColor": "#B23AF2", "pieOuterStrokeColor": "#B23AF2", "pieStrokeWidth": "2px"}}}%%
pie showData title Completion 93.75% (30h of 32h)
    "Completed Work (AI) #5B39F3" : 30
    "Remaining Work #FFFFFF" : 2
```

**Overall completion: `93.75%` (≈94%)** — computed strictly from AAP-scoped hours: `Completed / (Completed + Remaining) = 30 / (30 + 2) = 30/32 = 93.75%`.

| Metric | Hours |
|--------|-------|
| **Total Hours** | **32** |
| **Completed Hours (AI + Manual)** | **30** (30 AI + 0 Manual) |
| **Remaining Hours** | **2** |
| **Percent Complete** | **93.75%** |

### 1.3 Key Accomplishments

- ✅ **Answer document created** at the exact required path `blitzy/documentation/scapy_0925ada48540.md` (1,188 lines / 62,516 bytes; filename = source branch name).
- ✅ **Start stage documented** — canonical banner via `python3 -m scapy`, `scapy.VERSION = 2026.07.14`, two-stream stdout/stderr behavior, IPython/libpcap-absent caveats.
- ✅ **Build stage documented** — `/` layer stacking, selective `__repr__`, `summary()`/`command()`, the `show()`↔`show2()` (pre-build vs post-build) contrast, and the 54-byte `hexdump()`.
- ✅ **Send stage documented** — routing table + `(iface, output_ip, gateway_ip)` resolution, **both** L3 `send()` and L2 `sendp()` paths, and the `Sent 1 packets.` progress report.
- ✅ **Sniff stage documented** — live `AsyncSniffer` capture on loopback, dissection/reconstruction into typed layers, built-vs-dissected `repr` contrast, and `raw(captured) == raw(built)` proof.
- ✅ **Environment honesty** — `libpcap` absent (BPF compile fails; native `AF_PACKET` capture still works) and IPython absent are documented with the observed error text.
- ✅ **Grounding** — 87 unique `file:line` citations (118 occurrences); 100% resolve to the claimed symbol.
- ✅ **Repository integrity** — zero source modifications; working tree clean; temporary scripts removed.

### 1.4 Critical Unresolved Issues

| Issue | Impact | Owner | ETA |
|-------|--------|-------|-----|
| _None._ No unresolved compilation errors, failing tests, or missing functionality. The deliverable passed all autonomous validation gates with no corrections required. | None | — | — |

### 1.5 Access Issues

| System/Resource | Type of Access | Issue Description | Resolution Status | Owner |
|-----------------|----------------|-------------------|-------------------|-------|
| _No access issues identified._ The task ran with the required `root` (uid 0) privileges, the Scapy source tree was fully accessible, and the `lo`/`eth0` interfaces were present. No repository permissions, service credentials, or third-party API access were required. | — | — | Resolved | — |

### 1.6 Recommended Next Steps

1. **[High]** Perform a subject-matter-expert acceptance read-through of `blitzy/documentation/scapy_0925ada48540.md`, confirming every named prompt item is answered to satisfaction.
2. **[Medium]** Independently re-run reproduction Appendices B–E on the target host (as root) to confirm the structural values reproduce.
3. **[Low]** Merge the branch and confirm repository integrity (`git status --porcelain` reports only the answer document).
4. **[Low]** Optionally confirm the internal host IPs (`10.236.0.*`) shown as observed output are acceptable to publish externally.
5. **[Low]** If the Scapy source is ever upgraded past commit `0925ada`, re-verify the 87 `file:line` citations against the new line numbers.

---

## 2. Project Hours Breakdown

### 2.1 Completed Work Detail

| Component | Hours | Description |
|-----------|-------|-------------|
| Environment & startup investigation | 2 | §1 — two-stream banner (stdout 17-byte prompt / stderr banner), `scapy.VERSION 2026.07.14`, `conf` defaults, IPython/libpcap/cryptography caveats, Appendix A. |
| Build-stage investigation | 5 | §2 — `/` operator stacking, selective `__repr__`, `summary()`/`command()`, `show()` vs `show2()` incl. `post_build` computation, `hexdump()`. |
| Send-stage investigation | 5 | §3 — routing-table render, `(iface, output_ip, gateway_ip)` resolution, L3 `send()` + L2 `sendp()`, `__gen_send()` transmission report, interface selection. |
| Sniff-stage investigation | 5 | §4 — `AsyncSniffer` on `lo`, frame-count analysis, dissection pipeline, layer reconstruction, built-vs-dissected `repr`, libpcap-absent BPF + adversarial `lfilter`. |
| Cross-module source analysis & citation grounding | 3 | 87 unique `file:line` references across 17 REFERENCE modules; cause→effect reasoning per claim. |
| Answer-document authoring | 6 | 1,188-line evidence-grounded synthesis, §5 summary, determinism note (deterministic vs environment-dependent values). |
| Reproduction scripts & run-first execution | 2 | Appendices B–E scripts (build/send/sniff/bpf + adversarial), executed via the real API; ANSI-stripped capture. |
| QA / code-review remediation | 1 | Two commits addressing code-review findings and 3 MAJOR QA findings prior to final validation. |
| Final validation | 1 | Byte-for-byte reconciliation, 100% citation resolution, repository-integrity verification. |
| **Total Completed** | **30** | Sum of all completed components (validation: matches Completed Hours in §1.2). |

### 2.2 Remaining Work Detail

| Category | Hours | Priority |
|----------|-------|----------|
| Human SME acceptance review of the answer document (read-through; confirm all named prompt items answered) | 1.0 | High |
| Independent re-run of reproduction Appendices B–E on target host (confirm reproducibility) | 0.5 | Medium |
| Merge/deliver the document & confirm final repository integrity (`git status` clean) | 0.5 | Low |
| **Total Remaining** | **2.0** | (validation: matches Remaining Hours in §1.2 and §7 pie) |

> **Cross-check:** Section 2.1 (30) + Section 2.2 (2) = **32** = Total Project Hours in §1.2. ✅

---

## 3. Test Results

For this **read-only documentation** task, "tests" are Blitzy's autonomous **run-first reproduction** of the observed code paths plus static verification gates (citation resolution and source compilation). The AAP explicitly excludes adding a unit-test suite; all results below originate from Blitzy's autonomous validation logs for this project and were corroborated during this assessment.

| Test Category | Framework | Total Tests | Passed | Failed | Coverage % | Notes |
|---------------|-----------|-------------|--------|--------|-----------|-------|
| Build-path reproduction | Scapy-from-source + automated diff | 9 | 9 | 0 | 100% | repr progression, `overloaded_fields`, `IP.fields`, `summary`/`command`, `show` vs `show2` (ihl=5, len=40, chksum 0x7ccd/0x917c, dataofs=5), hexdump len=54 — all MATCH. |
| Send-path reproduction | Scapy-from-source + diff/MD5 | 8 | 8 | 0 | 100% | routing table, 2 route tuples `('lo','127.0.0.1','0.0.0.0')`, 4 byte-counts (L3=18, L2=18, count3=20, verb0=0), stderr MD5 identical ×3. |
| Sniff-path reproduction | AsyncSniffer on `lo` + automated diff | 8 | 8 | 0 | 100% | 2 loopback frames `[54,54]`, repr `<Sniffed: TCP:2 ...>`, `sniffed_on=lo`, ports (20,80), 2 checksums, `raw(r0)==raw(pkt)`, Appendix D diff. |
| libpcap-absent BPF + adversarial | Scapy native `AF_PACKET` | 4 | 4 | 0 | 100% | BPF error on stderr, native capture returns 2 frames, fail-closed `lfilter` assert, adversarial 6-vs-2 verified. |
| Citation resolution | Source grounding (grep/sed) | 87 | 87 | 0 | 100% | 87 unique `file:line` refs (118 occurrences) resolve to the claimed symbol; 15 independently re-verified this session. |
| Source compilation | `python -m py_compile` | 17 | 17 | 0 | 100% | All cited REFERENCE modules compile clean (exit 0). |
| **Totals** | — | **133** | **133** | **0** | **100%** | All autonomous run-first reproduction and static gates passed; no corrections required. |

---

## 4. Runtime Validation & UI Verification

Scapy has **no graphical UI**; its "interface" is its terminal/console presentation, which *is* the subject of the investigation. All output was verified ANSI-stripped for readability.

**Runtime health**
- ✅ **Operational** — Scapy imports and runs from the source tree (`import scapy.all`); `scapy.VERSION = 2026.07.14`.
- ✅ **Operational** — Canonical console entry point `python3 -m scapy` prints the banner and drops to a `>>>` prompt (stdout = exactly the 17-byte colored prompt; full banner on stderr).
- ✅ **Operational** — Native Linux `AF_PACKET` sockets selected (`L3PacketSocket`, `L2Socket`, `L2ListenSocket`).

**Build display surfaces (terminal "UI")**
- ✅ **Operational** — `__repr__` (selective field display), `summary()`, `command()`.
- ✅ **Operational** — `show()` (pre-build; computed fields `None`) vs `show2()` (post-build; `ihl=5`, `len=40`, IP chksum `0x7ccd`, TCP chksum `0x917c`, `dataofs=5`).
- ✅ **Operational** — `hexdump()` renders the assembled 54-byte wire form.

**Send / routing**
- ✅ **Operational** — routing table render and `(iface, output_ip, gateway_ip)` resolution.
- ✅ **Operational** — L3 `send()` and L2 `sendp()` both transmit; progress = `.` per packet then `Sent 1 packets.`

**Sniff / dissection**
- ✅ **Operational** — `AsyncSniffer` on loopback captures the transmitted packet; raw bytes reconstructed into typed `Ether/IP/TCP` layers; `raw(captured) == raw(built)`.
- ⚠ **Partial (environmental, documented)** — BPF `filter=` compilation fails because `libpcap` is absent (`ERROR: Cannot set filter: libpcap is not available. Cannot compile filter !`); native capture **still succeeds**, and an exact-signature user-space `lfilter=` provides fail-closed correlation.

---

## 5. Compliance & Quality Review

Cross-mapping of AAP deliverables and governing rules to Blitzy's quality/compliance benchmarks. Fixes for code-review and 3 MAJOR QA findings were applied by the authoring agents (commits `77d63f72`, `24096b4a`) **before** final validation; the Final Validator then found **no** further discrepancies.

| Benchmark / AAP Deliverable | Status | Progress | Evidence |
|-----------------------------|--------|----------|----------|
| Read-only source mandate (no `scapy/*` modified) | ✅ Pass | 100% | `git diff --name-status 0925ada..HEAD` = 1 entry (the `.md` only). |
| Single `.md` deliverable at exact path (name = branch) | ✅ Pass | 100% | `blitzy/documentation/scapy_0925ada48540.md` present. |
| Rule 1 — run-first investigation | ✅ Pass | 100% | Appendices B–E reproduction scripts; core build re-reproduced this session. |
| Rule 2 — exhaustive condition coverage + actual output | ✅ Pass | 100% | All display variants, **both** send paths, both before/after contrasts, unedited output. |
| Rule 3 — observed-output discipline + inferred labels | ✅ Pass | 100% | 13 "Direct answer" anchors; 2 explicit "inferred" labels (§5.1 no-route edge). |
| Rule 4 — grounded `file:line` citations + cause→effect | ✅ Pass | 100% | 87 unique citations; 100% resolve to claimed symbol. |
| MainRule/§0.8 — honest environment caveats | ✅ Pass | 100% | libpcap-absent (§4.5) and IPython-absent (§1) documented with observed error. |
| Cleanup — temporary scripts removed; repo clean | ✅ Pass | 100% | `git status --porcelain` empty; no `/tmp/obs_*.py` remain. |
| Zero placeholders / TODO / FIXME | ✅ Pass | 100% | Content scan returns none. |
| Balanced code fences | ✅ Pass | 100% | 64 fences = 32 balanced pairs. |

---

## 6. Risk Assessment

Overall posture: **LOW** — a read-only, loopback-only investigation whose sole artifact is a static Markdown file modifying no source.

| Risk | Category | Severity | Probability | Mitigation | Status |
|------|----------|----------|-------------|------------|--------|
| Environment-dependent surface values (routing IPs, Python version, rotating banner tagline, service names) differ off-host | Technical | Low | Medium | Document explicitly delineates deterministic (structural) vs environment-dependent values | Documented / Mitigated |
| Citation line-number drift if Scapy source is upgraded past commit `0925ada` | Technical | Low | Low | Exact commit stated; citations verified against it | Mitigated |
| Raw-socket / `root` requirement for reproduction | Security | Low | Low | Loopback-only (no egress); read-only observation scripts under `/tmp`, removed after use | Mitigated |
| Internal host IPs (`10.236.0.*`) and repo path embedded as observed output | Security | Low (Informational) | — | Non-sensitive; reviewer may confirm acceptable to publish | Accepted / Documented |
| No production runtime/service/monitoring (static document) | Operational | None (Informational) | — | N/A by design — deliverable is a document, not a service | N/A |
| `libpcap` absent → BPF compile fails (native capture still works) | Operational | Low | Medium | §4.5 documents the error + user-space `lfilter` fallback | Documented |
| No external integrations / API keys / network egress (loopback only) | Integration | None | — | N/A — task consumes Scapy's existing runtime only | N/A |
| Reproducibility depends on optional-package presence (IPython / libpcap / cryptography) | Integration | Low | Medium | Document enumerates exact package presence/absence | Documented |

---

## 7. Visual Project Status

```mermaid
%%{init: {"theme": "base", "themeVariables": {"pie1": "#5B39F3", "pie2": "#FFFFFF", "pieStrokeColor": "#B23AF2", "pieOuterStrokeColor": "#B23AF2", "pieStrokeWidth": "2px"}}}%%
pie showData title Project Hours Breakdown (Total 32h)
    "Completed Work #5B39F3" : 30
    "Remaining Work #FFFFFF" : 2
```

**Remaining hours by category (from §2.2) — total = 2h:**

```mermaid
%%{init: {"theme": "base", "themeVariables": {"pie1": "#B23AF2", "pie2": "#A8FDD9", "pie3": "#FFFFFF", "pieStrokeColor": "#5B39F3", "pieStrokeWidth": "2px"}}}%%
pie showData title Remaining Work by Category (2h)
    "SME acceptance review (High)" : 1.0
    "Independent reproduction (Medium)" : 0.5
    "Merge / repo-integrity (Low)" : 0.5
```

> **Integrity check:** "Remaining Work" = **2h** in the pie above, equal to §1.2 Remaining Hours (2h) and the §2.2 Hours sum (1.0 + 0.5 + 0.5 = 2.0). ✅

---

## 8. Summary & Recommendations

**Achievements.** The project delivered a complete, evidence-grounded answer document that follows one `Ether()/IP()/TCP()` packet end-to-end and documents exactly what Scapy shows at each stage — start, build, send, and sniff — with the actual observed output and a `file:line` citation behind every claim. All 25 AAP-scoped requirements are COMPLETED, including both required send paths (L3 `send()` / L2 `sendp()`) and both required contrasts (`show()` vs `show2()`; built vs dissected `repr`). Autonomous validation confirmed the document byte-for-byte with **no corrections required**.

**Remaining gaps.** None functionally. The outstanding **2 hours** are human-in-the-loop acceptance only: an SME read-through (1h), an independent reproduction re-run (0.5h), and merge/repo-integrity confirmation (0.5h). There is no build, deploy, or CI path-to-production for a standalone Markdown deliverable (the AAP explicitly excludes testing/CI/build/dependency changes).

**Critical path to production.** SME acceptance review → optional independent reproduction → merge. Because the working tree is already clean and the document is fully validated, this path is short and low-risk.

**Success metrics.** 87/87 citations resolve (100%); 133/133 autonomous checks pass (100%); 0 source files modified; working tree clean; core build path independently re-reproduced during this assessment (exact match).

**Production-readiness assessment.** The project is **93.75% complete** and, in the judgment of the autonomous validation, **production-ready pending human acceptance**. The document is complete, accurate, fully grounded in observed runtime output, reproducible, and leaves the repository in the exact intended state (only the answer document added).

| Metric | Value |
|--------|-------|
| AAP requirements completed | 25 / 25 |
| Completion (hours-based) | 93.75% (30h / 32h) |
| Autonomous checks passed | 133 / 133 (100%) |
| Citations resolved | 87 / 87 (100%) |
| Source files modified | 0 |
| Working tree | Clean |

---

## 9. Development Guide

How to run Scapy from source, reproduce the documented observations, and verify repository integrity. All commands below were executed and verified in the target environment (Linux, Python 3.13.7, root).

### 9.1 System Prerequisites

- **OS:** Linux (native `AF_PACKET` backend). The `lo` (127.0.0.1) and `eth0` interfaces present.
- **Python:** `>=3.7, <4` (observed **3.13.7**).
- **Privileges:** **root (uid 0)** — required for raw-socket `send`/`sendp`/`sniff`.
- **Source tree:** run Scapy from the checked-out repository, **not** an installed pip release.

### 9.2 Environment Setup

```bash
# From the repository root (branch already checked out at HEAD, base commit 0925ada):
cd /tmp/blitzy/scapy/blitzy-25029afa-3880-4f96-a5df-88edf3ddbb3d_d58022

# Confirm interpreter and privileges
python3 --version         # -> Python 3.13.7  (satisfies >=3.7,<4)
id -u                     # -> 0  (root; required for raw sockets)
```

- **No build/compile step** — Scapy is pure Python.
- **Optional packages:** IPython and `libpcap` are intentionally **absent** here (standard shell + BPF-compile-skipped-with-native-capture); `cryptography` is present (emits a harmless TripleDES deprecation warning). None of these block the investigation.

### 9.3 Dependency Installation

Scapy core has **no mandatory runtime dependencies** (`pyproject.toml` `[project]` declares none), so nothing needs to be installed to run from source. Optional extras only if you want them:

```bash
# OPTIONAL — richer console / crypto layers / docs (not required for this task)
pip install --break-system-packages ipython          # cli extra
pip install --break-system-packages 'cryptography>=2.0'  # all extra (already present)
```

### 9.4 Running & Reproducing

```bash
# Interactive console (prints banner + version, then a >>> prompt)
echo "exit()" | python3 -m scapy

# Library import + version check (run from repo root so source is on sys.path)
python3 -c "import sys; sys.path.insert(0,'.'); import scapy; print(scapy.VERSION)"
# -> 2026.07.14

# Reproduce the observations: copy each script from the answer document's
# Appendices B-E into /tmp (OUTSIDE the repo) and run as root:
python3 -B -u /tmp/obs_build.py       # build-stage output
python3 -B -u /tmp/obs_send.py        # send-stage output (L3 + L2)
python3 -B -u /tmp/obs_sniff.py       # sniff-stage output (AsyncSniffer on lo)
python3 -B -u /tmp/obs_sniff_bpf.py   # libpcap-absent BPF behavior + lfilter fallback

# Clean up afterward so the repository stays unchanged:
rm -f /tmp/obs_build.py /tmp/obs_send.py /tmp/obs_sniff.py /tmp/obs_sniff_bpf.py
```

### 9.5 Verification

```bash
# Build reproduction — expect an EXACT match to the document:
python3 - <<'PY'
import sys, re
sys.path.insert(0, '.')
from scapy.all import Ether, IP, TCP, raw
strip = lambda s: re.sub(r'\x1b\[[0-9;]*m', '', s)
pkt = Ether()/IP()/TCP()
print("repr   :", strip(repr(pkt)))               # <Ether  type=IPv4 |<IP  frag=0 proto=tcp |<TCP  |>>>
print("summary:", strip(pkt.summary()))           # Ether / IP / TCP 127.0.0.1:ftp_data > 127.0.0.1:http S
p2 = Ether(raw(pkt))
print("len    :", len(raw(pkt)))                  # 54
print("IP.chksum : 0x%04x" % p2[IP].chksum)       # 0x7ccd
print("TCP.chksum: 0x%04x" % p2[TCP].chksum)      # 0x917c
print("ihl/len/dataofs:", p2[IP].ihl, p2[IP].len, p2[TCP].dataofs)  # 5 40 5
PY

# Repository integrity — expect an empty (clean) working tree and one added file:
git status --porcelain                                   # (empty)
git diff --name-status 0925ada..HEAD                     # A  blitzy/documentation/scapy_0925ada48540.md
```

### 9.6 Troubleshooting (common error cases)

- **`ERROR: Cannot set filter: libpcap is not available. Cannot compile filter !`** — **Expected** when `libpcap` is absent. Native `AF_PACKET` capture still works; use an exact-signature `lfilter=` (user-space) for correlation instead of a BPF `filter=`.
- **`WARNING: IPython not available. Using standard Python shell instead.`** — **Expected** when IPython is absent; the console falls back to the standard Python REPL (AutoCompletion/History disabled).
- **`Operation not permitted` / socket errors** — you are not root; re-run as **uid 0**.
- **Wrong Scapy version or `ModuleNotFoundError`** — ensure the repository root is on `sys.path` (run from the repo root, or `sys.path.insert(0,'.')`); do not rely on an installed pip release.
- **`CryptographyDeprecationWarning: TripleDES ...`** — harmless version-skew warning from the present `cryptography` library at import of `scapy/layers/ipsec.py`; it does not affect any packet output.

---

## 10. Appendices

### A. Command Reference

| Command | Purpose |
|---------|---------|
| `echo "exit()" \| python3 -m scapy` | Launch console; print banner + `Version 2026.07.14`; exit. |
| `python3 -c "import sys; sys.path.insert(0,'.'); import scapy; print(scapy.VERSION)"` | Print Scapy version from the source tree. |
| `python3 -B -u /tmp/obs_build.py` | Reproduce build-stage output. |
| `python3 -B -u /tmp/obs_send.py` | Reproduce send-stage output (L3 `send` + L2 `sendp`). |
| `python3 -B -u /tmp/obs_sniff.py` | Reproduce sniff-stage output (`AsyncSniffer` on `lo`). |
| `python3 -B -u /tmp/obs_sniff_bpf.py` | Reproduce libpcap-absent BPF behavior + `lfilter` fallback. |
| `python3 -m py_compile scapy/*.py` | Compile-check cited REFERENCE modules. |
| `git status --porcelain` | Verify a clean working tree. |
| `git diff --name-status 0925ada..HEAD` | Confirm only the answer document was added. |

### B. Port Reference

Not applicable — no network service is started and no port is listened on. All transmission is confined to the **loopback** interface (`lo`, 127.0.0.1); the `Ether()/IP()/TCP()` packet's default TCP ports are `sport=20` (ftp_data) → `dport=80` (http), used only as packet field values, not as bound sockets.

### C. Key File Locations

| Path | Role |
|------|------|
| `blitzy/documentation/scapy_0925ada48540.md` | **The deliverable** — answer document (1,188 lines). |
| `scapy/packet.py` | REFERENCE — `__div__` L596, `__repr__` L552, `build`/`post_build`, `show` L1459 / `show2` L1473, `summary` L1642, `command` L1662, `dissect` L1049, `guess_payload_class` L1062. |
| `scapy/sendrecv.py` | REFERENCE — `send` L422, `sendp` L452, `__gen_send` L332, `_interface_selection` L616, `sniff` L1308, `AsyncSniffer` L981. |
| `scapy/route.py` | REFERENCE — `Route.__repr__` L47, `Route.route` L146. |
| `scapy/layers/inet.py` | REFERENCE — `IP`, `TCP`, `IP.route` L559, `bind_layers` L1101/L1113. |
| `scapy/layers/l2.py` | REFERENCE — `Ether` layer + defaults. |
| `scapy/config.py` | REFERENCE — `conf` defaults; Linux socket selection L647/L655/L656. |
| `scapy/arch/linux.py` | REFERENCE — `L3PacketSocket`, `L2Socket`, `L2ListenSocket`, `recv_raw`. |
| `scapy/utils.py`, `scapy/compat.py`, `scapy/plist.py`, `scapy/interfaces.py`, `scapy/supersocket.py`, `scapy/fields.py`, `scapy/base_classes.py`, `scapy/themes.py`, `scapy/main.py` | REFERENCE — display, serialization, capture-list, socket/factory, field rendering, metaclass, theming, entry point. |

### D. Technology Versions

| Component | Version / State |
|-----------|-----------------|
| Python | 3.13.7 (constraint `>=3.7, <4`) |
| Scapy | `2026.07.14` (run from source) |
| Repository commit (base) | `0925ada485406684174d6f068dbd85c4154657b3` |
| Branch HEAD | `24096b4a` |
| cryptography | 43.0.0 (present; TripleDES deprecation warning) |
| IPython | absent (standard Python shell used) |
| libpcap | absent (native `AF_PACKET` capture used) |
| Socket backend | `L3PacketSocket` / `L2Socket` / `L2ListenSocket` (Linux `AF_PACKET`) |

### E. Environment Variable Reference

No environment variables are required to run the investigation. Behavior is governed by Scapy's default configuration (`conf`), observed as:

| Setting | Value | Effect |
|---------|-------|--------|
| `conf.verb` | `2` | Verbosity — enables `.` per-packet progress and `Sent N packets.` |
| `conf.iface` | `eth0` | Default send interface (L2 `sendp` default; L3 `send` is route-driven). |
| `conf.color_theme` | `NoTheme` (piped) / colored (interactive) | ANSI coloring of `repr`/`show`/banner. |
| `conf.L3socket` / `conf.L2socket` / `conf.L2listen` | `L3PacketSocket` / `L2Socket` / `L2ListenSocket` | Native Linux socket selection. |

### F. Developer Tools Guide

- **Reproduction scripts (Appendices B–E of the answer document):** self-contained Python programs that prepend the repo root to `sys.path`, strip ANSI escapes, and write nothing into the repository. Keep them under `/tmp` and remove them after use.
- **ANSI stripping:** `re.sub(r'\x1b\[[0-9;]*m', '', s)` renders colored output as plain text identical to what the document shows.
- **Two-stream awareness:** the console banner is written to **stderr**; only the interactive prompt is on **stdout**. Separate them with `2>&1 >/dev/null` (banner) vs `2>/dev/null` (prompt) when capturing.
- **Adversarial correlation check:** without an exact `lfilter`, a loopback capture is overbroad (6 frames observed); the exact-signature `lfilter` narrows it fail-closed to the 2 correlated frames.

### G. Glossary

| Term | Meaning |
|------|---------|
| **Build** | Constructing a packet in memory via the `/` layer-stacking operator (`Ether()/IP()/TCP()`). |
| **`show()` vs `show2()`** | `show()` displays the in-memory packet (computed fields `None`); `show2()` serializes to bytes (runs `post_build`, resolving checksums/lengths) then re-dissects and displays. |
| **Overloaded field** | A field value set implicitly by a `bind_layers()` binding (e.g., `Ether.type=IPv4`, `IP.proto=tcp`, `IP.frag=0`) rather than by the user. |
| **Dissection** | Reconstructing typed layers from raw captured bytes via `dissect()` + `guess_payload_class()` against the `bind_layers()` registry. |
| **L3 vs L2 send** | `send()` (layer 3) is route-driven and prepends no Ethernet header choice logic; `sendp()` (layer 2) sends the frame as-is on a given interface. |
| **BPF** | Berkeley Packet Filter — kernel-level capture filter compiled via `libpcap`; unavailable here, so a user-space `lfilter` is used instead. |
| **`AF_PACKET`** | The native Linux raw-socket family Scapy uses for send/capture when `libpcap` is absent. |
| **AAP** | Agent Action Plan — the governing specification for this task. |

---

*End of Blitzy Project Guide. All hour figures (Total 32h, Completed 30h, Remaining 2h) and the completion percentage (93.75%) are consistent across Sections 1.2, 2.1, 2.2, 7, and 8. Colors: Completed = `#5B39F3`, Remaining = `#FFFFFF`.*