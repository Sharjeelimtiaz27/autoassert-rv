# AutoAssert-RV

**Formal Security Assertion Quality Ranking and Hardware Trojan Evaluation for RISC-V Processors**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![Status: Research](https://img.shields.io/badge/status-research-orange.svg)]()

> **Note on naming:** A DATE 2026 paper also uses the name "AutoAssert."
> Final system name pending advisor confirmation. This repository may be renamed.

---

## Overview

**AutoAssert-RV** is a five-stage formal pipeline that ranks the security
quality of SVA assertions for RISC-V processors, validates them against
a hardware Trojan benchmark, and refines them using formal gap findings.

- **Validate pre-translated assertions (AVS)** — QuestaSim compile check +
  JasperGold FPV (Proven + non-vacuous) on clean Ibex RTL. Input: SVA bind
  files from ISCAS article + SecMetric journal. No automated translation.
- **Rank assertion quality formally (AQRS)** — Compute the Assertion Quality
  Score (AQS) for each assertion using five independent dimensions:
  SRTLC, AER, SAPC, TCFC, and CWE/CVSS severity. Produces gap_report.json.
- **Evaluate against Trojans (TES)** — Run JasperGold FPV against 39
  RV-Trojan benchmark files across 6 TrustHub attack categories.
  Validates the AQRS ranker: high-AQS assertions should detect more Trojans.
- **Refine assertions (ARS)** — Python template script generates new assertions
  from gap_report.json (AER gaps, SAPC blind spots, TCFC missing categories).
  No LLM involvement. New assertions added to bind files; originals unchanged.
- **Validate improvement (RAVS)** — Same 39 Trojans re-tested against improved
  bind files. TDER, WTDR, AAD improvement reported vs Stage 3 (TES).

This repository accompanies the paper:
> **"AutoAssert-RV: Formal Security Assertion Quality Ranking and
> Trojan Evaluation for RISC-V Processors"** *(title draft — pending advisor)*
> S. Imtiaz, U. Reinsalu, T. Ghasempouri
> Tallinn University of Technology — Grant PSG837
> Target: IEEE Transactions on Very Large Scale Integration (TVLSI)
> Backup: IEEE Access

---

## Five-Stage Pipeline

```
AVS → AQRS → TES → ARS → RAVS
 10    66     39    0     39   = 154 JasperGold runs total
```

| Stage | Full Name | Abbreviation | Tool | Runs |
|-------|-----------|--------------|------|------|
| 1 | Assertion Validation Stage | AVS | QuestaSim + JasperGold FPV | 10 |
| 2 | Assertion Quality Ranking Stage | AQRS | JasperGold FPV + SPV | 66 |
| 3 | Trojan Evaluation Stage | TES | JasperGold FPV | 39 |
| 4 | Assertion Refinement Stage | ARS | Python template script | 0 |
| 5 | Refined Assertion Validation Stage | RAVS | JasperGold FPV | 39 |

---

## Key Features

### Stage 1 — Assertion Validation Stage (AVS)
Input: pre-translated SVA bind files from ISCAS article + SecMetric journal
(stored in `assertions/source_sva/`).
Validation: QuestaSim compile check → JasperGold FPV Proven + non-vacuous
on clean Ibex RTL. COV files collected for SRTLC in Stage 2.

Metric produced: **Security Assertion Translation Rate (SATR)**
`SATR = validated assertions / total source assertions × 100%`

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
├── CLAUDE.md                    ← Claude Code reads this every session (gitignored)
├── pipeline/
│   └── refine_assertions.py     ← Stage 4 ARS: Python template script only
│                                   (no translation scripts — advisor decision May 2026)
├── rtl/
│   └── ibex/                    ← processor-scoped (add rtl/cva6/ for future)
│       ├── original/            ← full Ibex RTL (33 files — parse 9 security modules)
│       ├── trojaned_rtl/        ← integrated trojaned files per module per category
│       └── generated_trojans/   ← RV-TroGen intermediate snippets (provenance)
├── assertion_dataset/           ← source assertion CSV files (NS31A + our own)
├── assertions/
│   ├── source_sva/              ← input: pre-translated SVA from ISCAS + SecMetric
│   ├── translated/              ← 10 verified bind files (AVS output — Stage 1)
│   └── refined/                 ← 10 improved bind files (ARS output — Stage 4)
├── jasper/
│   ├── fpv_run.tcl              ← AVS + TES + RAVS FPV template
│   ├── fpv_aer.tcl              ← AQRS AER FPV oracle template
│   ├── fpv_tcfc.tcl             ← AQRS TCFC category oracle template
│   ├── spv_sapc.tcl             ← AQRS SAPC SPV template (novel)
│   ├── run_avs.sh               ← Run Stage 1 AVS (10 runs)
│   ├── run_aqrs.sh              ← Run Stage 2 AQRS (66 runs)
│   ├── run_tes.sh               ← Run Stage 3 TES (39 runs)
│   ├── run_ravs.sh              ← Run Stage 5 RAVS (39 runs)
│   ├── oracles/                 ← 10 AER oracle modules (FPV)
│   └── tcfc_oracles/            ← 42 TCFC category oracle modules (FPV)
├── metrics/
│   ├── compute_satr.py          ← SATR from AVS validation logs
│   ├── compute_srtlc.py         ← SRTLC from Stage 1 COV files
│   ├── compute_aer.py           ← AER from AQRS oracle results
│   ├── compute_sapc.py          ← SAPC from AQRS SPV results (novel)
│   ├── compute_tcfc.py          ← TCFC from AQRS category results
│   ├── compute_aqs.py           ← AQS composite score
│   ├── compute_wtdr_aad.py      ← WTDR and AAD from TES/RAVS
│   ├── build_detection_matrix.py
│   └── plot_aqs_validation.py   ← AQS vs detection scatter plot
└── results/
    ├── step1/                   ← Stage 1 AVS: JasperGold FPV + COV files (10 runs)
    ├── step2/                   ← Stage 2 AQRS: AER + SAPC + TCFC output + gap_report.json (66 runs)
    ├── step3/                   ← Stage 3 TES: Trojan detection matrix (39 runs)
    ├── step4/                   ← Stage 4 ARS: gap report + new assertion candidates (Python only)
    ├── step5/                   ← Stage 5 RAVS: improved detection matrix (39 runs)
    └── metrics/                 ← Final computed metric tables (AQS, TDER, WTDR, AAD)
```

---

## Requirements

- Python 3.9+ with pandas (`pip install pandas`)
- QuestaSim (AVS compile validation — server)
- JasperGold with FPV + SPV licences (AVS, AQRS, TES, RAVS — server)
- Ibex RISC-V RTL (place in `rtl/ibex/original/`)

**No GPU required. No model training. No API cost.**

---

## Quick Start

```bash
git clone https://github.com/Sharjeelimtiaz27/autoassert-rv
cd autoassert-rv
pip install pandas

# Place pre-translated SVA bind files in assertions/source_sva/
# Place Ibex RTL in rtl/ibex/original/

# Stage 1 (AVS) — server: compile + JasperGold FPV on clean RTL
bash jasper/run_avs.sh

# Stage 2 (AQRS) — server: AER + SAPC + TCFC (66 JasperGold runs)
bash jasper/run_aqrs.sh

# Stage 3 (TES) — server: 39 Trojan detection runs
bash jasper/run_tes.sh

# Stage 4 (ARS) — laptop: generate improved assertions from gap report
python pipeline/refine_assertions.py --gap-report results/step2/gap_report.json

# Stage 5 (RAVS) — server: re-test improved assertions on same 39 Trojans
bash jasper/run_ravs.sh

# Compute all metrics
python metrics/compute_aqs.py
python metrics/build_detection_matrix.py
python metrics/plot_aqs_validation.py
```

---

## Laptop + Server Workflow

```bash
# Laptop — place input SVA files, compute Python-only metrics, run ARS
cp your_source_sva/*.sv assertions/source_sva/
python pipeline/refine_assertions.py --gap-report results/step2/gap_report.json
git add assertions/ results/step4/
git commit -m "ARS: refined assertions generated"
git push

# Server — all JasperGold runs (needs QuestaSim + JasperGold licences)
git pull
bash jasper/run_avs.sh   && git add results/step1/ && git commit -m "AVS complete"
bash jasper/run_aqrs.sh  && git add results/step2/ && git commit -m "AQRS complete"
bash jasper/run_tes.sh   && git add results/step3/ && git commit -m "TES complete"
bash jasper/run_ravs.sh  && git add results/step5/ && git commit -m "RAVS complete"
git push
```

---

## Comparison with Related Work

### Table 1 — SVA automation: generation and translation

| Feature | AutoAssert-RV | Transys [1] | Chuah MEMOCODE [2] | Kande et al. [3] | AssertLLM [4] | TrustAssert [5] | Assertain [6] |
|---------|--------------|-------------|-------------------|-----------------|--------------|----------------|--------------|
| Task | Validation + quality ranking | Translation | Manual writing | Generation | Generation | Generation | Generation |
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
@article{imtiaz2026autoassert,
  title   = {AutoAssert-RV: Formal Security Assertion Quality Ranking
             and Trojan Evaluation for RISC-V Processors},
  author  = {Imtiaz, Sharjeel and Reinsalu, Uljana and Ghasempouri, Tara},
  journal = {IEEE Transactions on Very Large Scale Integration (TVLSI)},
  year    = {2026}
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

**Version:** 1.3.0 | **Last Updated:** May 2026