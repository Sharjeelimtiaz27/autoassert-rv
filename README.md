# AutoAssert-RV

**Automated Security Assertion Translation and Formal Hardware Trojan Evaluation for RISC-V Processors**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![Status: Research](https://img.shields.io/badge/status-research-orange.svg)]()

> **Note on naming:** A DATE 2026 paper also uses the name "AutoAssert."
> Final system name pending advisor confirmation. This repository may be renamed.

---

## Overview

**AutoAssert-RV** is a five-stage automated pipeline for security assertion
translation, formal quality ranking, and hardware Trojan evaluation for
RISC-V processors.

- **Translate assertions automatically (ATS)** — Map expert security
  properties from NS31A (reference processor) to Ibex (target processor)
  using Claude Code CLI with QuestaSim compile validation
- **Rank assertion quality formally (AQRS)** — Compute the Assertion Quality
  Score (AQS) for each translated assertion using five independent dimensions:
  SRTLC, AER, SAPC, TCFC, and CWE/CVSS severity
- **Evaluate against Trojans (TES)** — Run JasperGold FPV against 39
  independent RV-Trojan files across 6 TrustHub attack categories
- **Refine assertions automatically (ARS)** — Python template script generates
  new assertions from AQS gap report without LLM involvement
- **Validate improvement (RAVS)** — Re-test improved assertions against the
  same 39 Trojans and compare detection results

This repository accompanies the paper:
> **"AutoAssert-RV: Automated Security Assertion Translation and Formal Trojan
> Evaluation for RISC-V Processors"**
> S. Imtiaz, U. Reinsalu, T. Ghasempouri
> Tallinn University of Technology — Grant PSG837
> Target: IEEE Transactions on Very Large Scale Integration (TVLSI)
> Backup: IEEE Access

---

## Five-Stage Pipeline

```
ATS → AQRS → TES → ARS → RAVS
 12    66     39    0     39   = 156 JasperGold runs total
```

| Stage | Full Name | Abbreviation | Tool | Runs |
|-------|-----------|--------------|------|------|
| 1 | Assertion Translation Stage | ATS | Claude Code CLI + QuestaSim + JasperGold FPV | 12 |
| 2 | Assertion Quality Ranking Stage | AQRS | JasperGold FPV + SPV | 66 |
| 3 | Trojan Evaluation Stage | TES | JasperGold FPV | 39 |
| 4 | Assertion Refinement Stage | ARS | Python template script | 0 |
| 5 | Refined Assertion Validation Stage | RAVS | JasperGold FPV | 39 |

---

## Key Features

### Stage 1 — Assertion Translation Stage (ATS)
Five sub-steps: RTL parsing (pyverilog) → signals.json → prompt builder →
Claude Code CLI translation → QuestaSim compile validation (max 3 retries)
→ JasperGold FPV Proven + non-vacuous gate.

Metric produced: **Translation Acceptance Rate (TAR)**
`TAR = translated assertions / total NS31A assertions × 100%`

### Stage 2 — Assertion Quality Ranking Stage (AQRS)
Computes five independent quality dimensions per assertion on CLEAN RTL only.
No Trojans involved. Produces AQS score and gap_report.json.

**Assertion Quality Score (AQS):**
`AQS = 0.20×CVSS_norm + 0.20×AER + 0.20×SRTLC + 0.20×SAPC + 0.20×TCFC`

| Component | Full Name | Tool | What it measures |
|-----------|-----------|------|-----------------|
| SRTLC | Security RTL Coverage | JasperGold FPV COV | % of NS31A-mapped security RTL inside assertion COIs |
| AER | Assertion Evasion Resistance | JasperGold FPV + oracle | Can any behavioral Trojan bypass this assertion? |
| SAPC | Security Assertion Path Coverage | JasperGold SPV | Are there structural input paths to security signals no assertion monitors? |
| TCFC | Trojan Category Formal Coverage | JasperGold FPV oracles | Does the assertion set formally cover all 6 TrustHub categories? |
| CVSS | From CWE mapping | Lookup table | How dangerous is the vulnerability this assertion guards? |

**Key theorem:** AER=1 does not guarantee SAPC=100%. Behavioral completeness
and structural completeness are orthogonal dimensions. Both must be measured.
SAPC is the first SPV-based assertion quality metric in the literature.

### Stage 3 — Trojan Evaluation Stage (TES)
39 RV-Trojan files tested against translated assertions.
Validates AQRS ranker: high-AQS assertions should detect more Trojans.

Metrics: TDER, WTDR (stealth-weighted), AAD (per-Trojan detection breadth)

### Stage 4 — Assertion Refinement Stage (ARS)
Python template script reads gap_report.json from AQRS.
Generates new SVA assertions for unmonitored signals identified by
AER, SAPC, and TCFC. No LLM involved — hardcoded SVA templates with
signal name substitution. New assertions added to bind files; existing
assertions never modified.

### Stage 5 — Refined Assertion Validation Stage (RAVS)
Same 39 RV-Trojans tested against improved bind files.
Reports improvement in TDER, WTDR, AAD vs Stage 3 (TES).

---

## RISC-V Hardware Trojan Benchmark (RV-Trojan)

39 trojaned RTL files for 9 security-critical Ibex modules.
Constructed from TrustHub taxonomy and RISC-V security literature.
Built independently of the assertion set — no circularity.

| # | Category | Mechanism |
|---|----------|-----------|
| 1 | Availability | Periodic duty-cycle stall on control signals |
| 2 | Covert | Timing channel encoding on single-bit output |
| 3 | Denial of Service | Permanent path disable on control enables |
| 4 | Integrity | XOR data corruption on output signals |
| 5 | Leak | Secret data routing to stable output ports |
| 6 | Privilege | Forced machine-mode escalation on privilege registers |

---

## Module Coverage

| Module | RTL File | Type | TrustHub categories |
|--------|----------|------|---------------------|
| PMP | `ibex_pmp.sv` | Combinational | Covert, DoS, Integrity, Leak, Privilege |
| CSR | `ibex_cs_registers.sv` | Sequential | All 6 |
| DO | `ibex_controller.sv` | Sequential | All 6 (shared file) |
| ETI | `ibex_controller.sv` | Sequential | All 6 (shared file) |
| CF | `ibex_controller.sv` | Sequential | All 6 (shared file) |
| MT | `ibex_controller.sv` | Sequential | All 6 (shared file) |
| MA | `ibex_load_store_unit.sv` | Sequential | Availability, Covert, DoS, Integrity, Leak |
| IE | `ibex_id_stage.sv` + `ibex_ex_block.sv` | Sequential | All 6 |
| RU | `ibex_wb_stage.sv` | Sequential | Availability, Covert, DoS, Integrity, Leak |

`ibex_controller.sv` covers 4 logical modules (DO, ETI, CF, MT) with 4
separate SVA bind files and one shared AER oracle.

---

## Repository Structure

```
autoassert-rv/
├── CLAUDE.md                    ← Claude Code reads this every session
├── pipeline/
│   ├── run_step1.py             ← ATS master orchestrator
│   ├── parse_rtl.py             ← ATS 1A: pyverilog RTL parser
│   ├── build_prompt.py          ← ATS 1B: prompt builder (seq/comb templates)
│   ├── translate.py             ← ATS 1C: Claude Code CLI translation
│   ├── validate_compile.py      ← ATS 1D: QuestaSim compile loop
│   ├── build_wrapper.py         ← ATS 1E: bind file builder
│   ├── validate_fpv.py          ← ATS 1F: JasperGold FPV baseline
│   ├── refine_assertions.py     ← ARS: Python template script (Stage 4)
│   ├── templates/
│   │   ├── sequential_prompt.txt
│   │   └── combinational_prompt.txt
│   ├── signals/                 ← MODULE_signals.json (gitignored)
│   ├── state/                   ← MODULE_state.json retry tracking
│   └── logs/                    ← MODULE_tar_log.json TAR metric
├── rtl/
│   ├── original/                ← 8 clean Ibex RTL files
│   └── trojaned/                ← 39 trojaned RTL files
├── assertion_dataset/           ← Source assertions in Excel (.xlsx), 10 files, ns31a_ prefix
├── assertions/
│   ├── translated/              ← 10 SVA bind files (ATS output)
│   └── refined/                 ← 10 improved bind files (ARS output)
├── jasper/
│   ├── fpv_run.tcl              ← ATS + TES + RAVS FPV template
│   ├── fpv_aer.tcl              ← AQRS AER FPV oracle template
│   ├── fpv_tcfc.tcl             ← AQRS TCFC category oracle template
│   ├── spv_sapc.tcl             ← AQRS SAPC SPV template (novel)
│   ├── run_ats.sh               ← Run Stage 1 baseline (12 runs)
│   ├── run_aqrs.sh              ← Run Stage 2 AQRS (66 runs)
│   ├── run_tes.sh               ← Run Stage 3 TES (39 runs)
│   ├── run_ravs.sh              ← Run Stage 5 RAVS (39 runs)
│   ├── oracles/                 ← 10 AER oracle modules (FPV)
│   └── tcfc_oracles/            ← 42 TCFC category oracle modules (FPV)
├── metrics/
│   ├── compute_tar.py           ← TAR and SAR from ATS logs
│   ├── compute_srtlc.py         ← SRTLC from AQRS COV files
│   ├── compute_aer.py           ← AER from AQRS oracle results
│   ├── compute_sapc.py          ← SAPC from AQRS SPV results
│   ├── compute_tcfc.py          ← TCFC from AQRS category results
│   ├── compute_aqs.py           ← AQS composite score
│   ├── compute_wtdr_aad.py      ← WTDR and AAD from TES/RAVS
│   ├── build_detection_matrix.py
│   └── plot_aqs_validation.py   ← AQS vs detection scatter plot
├── errors/
│   └── archive/                 ← All error logs (never deleted)
└── results/
    ├── ats/                     ← ATS FPV baseline results
    ├── aqrs/                    ← AQRS metric results + gap_report.json
    ├── tes/                     ← TES detection matrix
    ├── ravs/                    ← RAVS improved detection matrix
    └── metrics/                 ← Final computed metric tables
```

---

## Requirements

- Python 3.9+ with pyverilog (`pip install pyverilog`)
- Claude Code CLI (installed and authenticated — uses existing subscription)
- QuestaSim (ATS compile validation — server)
- JasperGold with FPV + SPV licences (AQRS, TES, RAVS — server)
- Ibex RISC-V RTL (place in `rtl/original/`)

**No GPU required. No model training. Zero extra API cost beyond Claude Code subscription.**

---

## Quick Start

```bash
git clone https://github.com/Sharjeelimtiaz27/autoassert-rv
cd autoassert-rv
pip install pyverilog

# Stage 1 (ATS) — Laptop: parse and translate
python pipeline/run_step1.py --module pmp --mode local
git push

# Stage 1 (ATS) — Server: compile and FPV
git pull && python pipeline/run_step1.py --module pmp --mode server

# All 9 modules
python pipeline/run_step1.py --all-modules

# Stage 2 (AQRS) — all 66 JasperGold runs
bash jasper/run_aqrs.sh

# Stage 3 (TES) — all 39 Trojan runs
bash jasper/run_tes.sh

# Stage 4 (ARS) — auto-generate improved assertions
python pipeline/refine_assertions.py --gap-report results/aqrs/gap_report.json

# Stage 5 (RAVS) — re-test improved assertions
bash jasper/run_ravs.sh

# Compute all metrics
python metrics/compute_aqs.py
python metrics/build_detection_matrix.py
python metrics/plot_aqs_validation.py
```

---

## Laptop + Server Workflow

```bash
# Laptop (parse + translate — no licences needed)
python pipeline/run_step1.py --module csr --mode local
git add pipeline/signals/ pipeline/logs/ assertions/translated/
git commit -m "ATS local: csr translated"
git push

# Server (QuestaSim + JasperGold)
git pull
python pipeline/run_step1.py --module csr --mode server
git add results/ errors/
git commit -m "ATS server: csr validated"
git push
```

---

## Comparison with Related Work

### Table 1 — SVA automation: generation and translation

| Feature | AutoAssert-RV | Transys [1] | Chuah MEMOCODE [2] | Kande et al. [3] | AssertLLM [4] | TrustAssert [5] | Assertain [6] |
|---------|--------------|-------------|-------------------|-----------------|--------------|----------------|--------------|
| Task | Translation + ranking | Translation | Manual writing | Generation | Generation | Generation | Generation |
| Year | 2025 | 2020 | 2023 | 2024 | 2025 | 2026 | 2025 |
| Reuses expert assertions | Yes | Yes | N/A | No | No | No | No |
| Formal backend | JasperGold FPV+SPV | None | JasperGold | Simulation | Simulation | FPV correct rate | Simulation |
| Trojan evaluation | Yes — 39 | None | None | None | None | None | None |
| Assertion quality ranker | AQS (5 components) | None | None | None | None | None | None |
| GPU required | No | No | No | No | No | Yes (16×V100) | No |
| Evaluation metric | JasperGold FPV+vacuity | None | FPV | BLEU/ROUGE | Syntax/FPV | BLEU/ROUGE | FPV |

### Table 2 — Hardware Trojan generation benchmark

| Feature | AutoAssert-RV | TrojanForge [7] | SENTAUR [8] | GHOST [9] | 0ena [10] | SoC-HTs [11] | Trust-Hub [12] |
|---------|--------------|----------------|------------|----------|----------|-------------|--------------|
| Year | 2025 | 2024 | 2024 | 2025 | 2024 | 2023 | 2008 |
| Target arch | RISC-V (Ibex) | ISCAS-85 | AES/UART | AES/UART | RISC-V (CVA6) | RISC-V | Multi-IP |
| Trojans | 39 | ~700 | ~17 | 14 | 23 | ~106 | ~106 |
| TrustHub categories | 6 (all) | 1 | 3 | 3 | 2 | 3 | 4 |
| Independent of assertions | Yes | N/A | N/A | N/A | N/A | N/A | N/A |
| E2E formal detection | Yes (JasperGold) | No | No | No | No | No | No |

### Table 3 — Formal hardware Trojan detection

| Feature | AutoAssert-RV | Chuah ISQED 2026 [13] | HT-PGFV [14] | Eslami ISQED 2022 [15] |
|---------|--------------|----------------------|-------------|----------------------|
| Year | 2025 | 2026 | 2024 | 2022 |
| Target | RISC-V Ibex (9 modules) | RISC-V 4-stage (PC only) | ISCAS/TrustHub | ISCAS/TrustHub |
| Assertion source | Translated from NS31A | Manual (296 properties) | Auto-generated SVA | Functional reuse |
| Generic HT model | AER (per-assertion score) | Yes (set completeness check) | No | No |
| Structural path coverage | SAPC (SPV — novel) | Connection properties (manual) | No | No |
| TrustHub categories | 6 (TCFC formal proof) | PC-focused only | Mixed | Mixed |
| Quality ranker | AQS (5 components) | None | None | None |

**References:**
[1] R. Zhang and C. Sturton, "Transys," IEEE S&P, 2020.
[2] C. S. Chuah et al., "Formal Verification of Security Properties," MEMOCODE, 2023.
[3] R. Kande et al., "Security Assertions by LLMs," IEEE TIFS, 2024.
[4] W. Fang et al., "AssertLLM," ASPDAC, 2025.
[5] Q. Zhai et al., "TrustAssert/AutoAssert," DATE, 2026.
[6] Assertain, arXiv:2604.01583, 2025.
[7] K. Hui et al., "TrojanForge," arXiv:2405.15184, 2024.
[8] J. Bhandari et al., "SENTAUR," arXiv:2407.12352, 2024.
[9] M. O. Faruque et al., "GHOST," Electronics, 2025.
[10] A. Moschos et al., "0ena," ACNS, 2024.
[11] S. Deb et al., "SoC-HTs," IEEE, 2023.
[12] Trust-Hub, "Hardware Trojan Benchmarks," trust-hub.org.
[13] C. S. Chuah et al., "Reliable HT Detection for RISC-V," ISQED, 2026.
[14] HT-PGFV, Electronics, 2024.
[15] Eslami et al., "Reusing Verification Assertions," ISQED, 2022.

---

## Ethical Use

This tool is intended for security research and academic evaluation only.
It must not be used to insert Trojans into production hardware.

---

## Acknowledgments

Built on the **lowRISC Ibex** RISC-V processor (Apache 2.0):
https://github.com/lowRISC/ibex

Trojan taxonomy based on **TrustHub** Hardware Trojan benchmarks:
https://trust-hub.org

Funded by the Estonian Research Council grant **PSG837**.

---

## Citation

```bibtex
@article{imtiaz2025autoassert,
  title   = {AutoAssert-RV: Automated Security Assertion Translation
             and Formal Trojan Evaluation for RISC-V Processors},
  author  = {Imtiaz, Sharjeel and Reinsalu, Uljana and Ghasempouri, Tara},
  journal = {IEEE Transactions on Very Large Scale Integration (TVLSI)},
  year    = {2025}
}
```

---

## License

MIT — see [LICENSE](LICENSE) file.

---

## Contact

- **Author:** Sharjeel Imtiaz, Early Stage Researcher
- **Institution:** TalTech, Estonia
- **Email:** sharjeel.imtiaz@taltech.ee
- **Repository:** https://github.com/Sharjeelimtiaz27/autoassert-rv

**Version:** 1.2.0 | **Last Updated:** April 2026