# AutoAssert-RV

**Automated Security Assertion Translation and Formal Hardware Trojan Evaluation for RISC-V Processors**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![Status: Research](https://img.shields.io/badge/status-research-orange.svg)]()

---

## Overview

**AutoAssert-RV** is a Claude-assisted pipeline for automated security assertion
translation and formal hardware Trojan evaluation for RISC-V processors.

- **Translate assertions automatically** — Map security properties from NS31A
  (reference) to Ibex (target) using an LLM-assisted pipeline with QuestaSim
  compile validation
- **Evaluate formal detection** — Run JasperGold FPV and SPV against 39
  hardware Trojans across 6 TrustHub attack categories
- **Measure detection quality** — Compute 4 novel security metrics (TAR, WTDR,
  AAD, AER) plus 4 established baselines (SATR, TDER, TPI, SAC)
- **Reproduce paper results** — Full reproducible experiment suite with 63
  JasperGold runs

This repository accompanies the paper:
> **"AutoAssert-RV: Automated Security Assertion Translation and Formal Trojan
> Evaluation for RISC-V Processors"**
> S. Imtiaz, U. Reinsalu, T. Ghasempouri
> Tallinn University of Technology — Grant PSG837
> Target: IEEE Access

---

## Key Features

### Claude-Assisted Translation Pipeline
Three-phase pipeline: RTL signal extraction (pyverilog) → Claude Code assertion
translation → QuestaSim compile validation (max 3 retries). Produces SVA bind
files per module. Measures Translation Automation Ratio (TAR).

### RISC-V Hardware Trojan Benchmark
39 trojaned RTL files for 9 security-critical Ibex modules.
Constructed following the TrustHub hardware Trojan taxonomy and RISC-V
security literature. One Trojan per file, one attack category per Trojan.

| # | Category | Target | Mechanism |
|---|----------|--------|-----------|
| 1 | Availability | Control enable signals | Periodic duty-cycle stall |
| 2 | Covert | Single-bit output | Timing channel encoding |
| 3 | Denial of Service | Control enable signals | Permanent path disable |
| 4 | Integrity | Data output signals | XOR data corruption |
| 5 | Leakage | Stable output ports | Secret data routing |
| 6 | Privilege | Privilege-mode registers | Forced M-mode escalation |

### Four Novel Security Metrics

| Metric | Formula | What it measures |
|--------|---------|-----------------|
| TAR | correct auto-mappings / total × 100% | Translation automation quality |
| WTDR | Σ(TPI×detected) / ΣTPI × 100% | Stealth-weighted detection rate |
| AAD | triggered assertions / module total × 100% | Per-Trojan detection breadth |
| AER | JasperGold SPV Proven/CEX | Formal evasion resistance |

### Three-Phase JasperGold Formal Verification
- **Phase 1:** FPV on original clean RTL (baseline — 12 runs)
- **Phase 2:** FPV on all 39 trojaned variants (detection matrix — 39 runs)
- **Phase 3:** SPV with parametric oracle (AER formal proof — 12 runs)
- **Total: 63 JasperGold runs** across 9 modules

---

## Module Coverage

| Module | RTL File | Security concern |
|--------|----------|-----------------|
| PMP | `ibex_pmp.sv` | Physical memory protection |
| CSR | `ibex_cs_registers.sv` | Control/status registers |
| DO | `ibex_controller.sv` | Debug operation |
| ETI | `ibex_controller.sv` | Exception/trap interface |
| CF | `ibex_controller.sv` | Control flow |
| MT | `ibex_controller.sv` | Mode transitions |
| MA | `ibex_load_store_unit.sv` | Memory access |
| IE | `ibex_id_stage.sv` + `ibex_ex_block.sv` | Instruction execution |
| RU | `ibex_wb_stage.sv` | Register writeback |

`ibex_controller.sv` covers 4 logical modules (DO, ETI, CF, MT) with 4
separate SVA bind files — one per module.

---

## Repository Structure

```
autoassert-rv/
├── pipeline/
│   ├── prompts/         Claude prompt templates (one per module)
│   ├── signals/         Extracted signal JSON (gitignored)
│   ├── parse_rtl.py     RTL parser → MODULE_signals.json
│   ├── translate.py     Claude-assisted assertion translator
│   ├── validate.py      QuestaSim compile validator (3-retry loop)
│   ├── run_pipeline.sh  Single-module runner
│   └── batch_all.sh     All 9 modules in sequence
├── rtl/
│   ├── original/        Clean Ibex RTL (9 modules)
│   └── trojaned/        39 trojaned RTL files (6 per module)
├── assertions/
│   ├── source/          NS31A reference assertions (SVA)
│   └── translated/      Ibex translated SVA bind files
├── jasper/
│   ├── constraints/     JasperGold constraint files per module
│   ├── oracles/         Parametric Trojan oracle modules (AER)
│   ├── results/         JasperGold output (gitignored)
│   ├── fpv_run.tcl      FPV master template (parameterised)
│   ├── spv_aer.tcl      SPV AER template
│   └── run_all.sh       Batch all 63 runs
├── metrics/
│   ├── outputs/         Metric tables + plots (gitignored)
│   ├── compute_tpi.py   TPI computation from trigger bits
│   ├── build_matrix.py  Detection matrix builder
│   ├── compute_metrics.py TAR, WTDR, AAD, AER computation
│   └── plot_tpi_aad.py  TPI vs AAD scatter plot
└── docs/
    ├── paper/           LaTeX source
    └── figures/         Paper figures
```

---

## Requirements

- Python 3.9+ with pyverilog (`pip install pyverilog`)
- Claude Code CLI (installed and authenticated)
- QuestaSim (Phase 2 compile validation)
- JasperGold with FPV + SPV licences (Phase 3)
- Ibex RISC-V RTL (place in `rtl/original/`)

---

## Quick Start

```bash
git clone https://github.com/Sharjeelimtiaz27/autoassert-rv
cd autoassert-rv

# Phase 1-2: Parse and translate (one module)
python pipeline/parse_rtl.py --input rtl/original/ibex_pmp.sv
bash pipeline/run_pipeline.sh pmp

# Phase 1-2: All 9 modules
bash pipeline/batch_all.sh

# Phase 3: All 63 JasperGold runs
bash jasper/run_all.sh

# Compute all metrics
python metrics/compute_tpi.py
python metrics/build_matrix.py
python metrics/compute_metrics.py
python metrics/plot_tpi_aad.py
```

---

## JasperGold Commands

```bash
# Phase 1 — FPV baseline (clean RTL)
MODULE=ibex_pmp BIND=ibex_pmp_bind jg -tcl jasper/fpv_run.tcl -no_gui

# Phase 2 — FPV trojaned RTL
MODULE=ibex_pmp BIND=ibex_pmp_bind \
  DUT=rtl/trojaned/ibex_pmp_trojan_DoS.sv \
  jg -tcl jasper/fpv_run.tcl -no_gui

# Phase 3 — SPV AER (evasion resistance)
MODULE=ibex_pmp PROP=PMP_1 jg -tcl jasper/spv_aer.tcl -no_gui

# Batch all 63 runs
bash jasper/run_all.sh
```

---

## Comparison with Related Work

### Table 1 — SVA automation: generation and translation

| Feature | AutoAssert-RV (Ours) | Transys [1] | Chuah et al. [2] | Kande et al. [3] | AssertLLM [4] | SV-LLM [5] | Assertain [6] |
|---------|---------------------|-------------|------------------|-----------------|--------------|------------|--------------|
| Task | Translation | Translation | Manual writing | Generation | Generation | Generation (multi-agent) | Generation |
| Year | 2026 | 2020 | 2023 | 2024 | 2025 | 2025 | 2025 |
| Input | Validated SVA + RTL | RTL + rules | RISC-V spec | RTL + NL | NL spec | SoC files | RTL + spec |
| Reuses validated assertions | **Yes** | Yes | N/A | No | No | No | No |
| Target | RISC-V Ibex | Generic | RISC-V NS31A | ISCAS | Generic RTL | Generic SoC | Generic RTL |
| Formal backend | JasperGold FPV+SPV | None | JasperGold | Simulation | Simulation | Simulation | Simulation |
| Trojan evaluation | **Yes — 39** | None | None | None | None | None | None |
| Novel metrics | TAR, WTDR, AAD, AER | None | None | None | None | None | None |
| Open source | Yes | Partial | No | No | No | No | No |

### Table 2 — Hardware Trojan generation benchmark

| Feature | AutoAssert-RV (Ours) | TrojanForge [7] | SENTAUR [8] | GHOST [9] | 0ena [10] | SoC-HTs [11] | Trust-Hub [12] |
|---------|---------------------|----------------|------------|----------|----------|-------------|--------------|
| Year | 2026 | 2024 | 2024 | 2025 | 2024 | 2023 | 2008 |
| Target arch | RISC-V (Ibex) | ISCAS-85 | AES/UART/RAM | AES/UART/SRAM | RISC-V (CVA6) | RISC-V (Ariane) | Multi-IP |
| Abstraction | RTL (SV) | Gate | RTL | RTL | Layout | RTL (AXI) | Mixed |
| Automation | Full | Full | Semi | Full | Manual | Manual | Manual |
| Trojans | 39 | ~700 | ~17 | 14 | 23 | ~106 | ~106 |
| Categories | **6 (TrustHub)** | 1 | 3 | 3 | 2 | 3 | 4 |
| TrustHub lineage | **Yes** | No | Partial | Partial | No | No | Source |
| RISC-V priv. spec | **Yes** | No | No | No | Yes | No | No |
| E2E simulation | **Yes (QuestaSim)** | No | No | No | No | No | No |
| Used for formal detection | **Yes (JasperGold)** | No | No | No | No | No | No |
| Open source | Yes (MIT) | Yes | No | Yes | Yes | Yes | Partial |

### Table 3 — Formal hardware Trojan detection

| Feature | AutoAssert-RV (Ours) | HT-PGFV [13] | QuEST [14] |
|---------|---------------------|-------------|-----------|
| Year | 2026 | 2024 | 2025 |
| Detection method | SVA bind files + JasperGold FPV | Auto-SVA + formal | Shannon entropy |
| Target | RISC-V processor | ISCAS / Trust-Hub | Generic IP |
| Trojan categories | 6 — all TrustHub | Mixed | Leakage only |
| RISC-V specific | Yes | No | No |
| Assertion metrics | WTDR, AAD, AER | None | Entropy-based |
| Evasion resistance proof | **AER (formal)** | None | None |

**References:**
[1] R. Zhang and C. Sturton, "Transys," IEEE S&P, 2020.
[2] C. S. Chuah et al., "Formal Verification of Security Properties on RISC-V Processors," MEMOCODE, 2023.
[3] R. Kande et al., "(Security) Assertions by Large Language Models," IEEE TIFS, 2024.
[4] W. Fang et al., "AssertLLM," ASPDAC, 2025.
[5] D. Saha et al., "SV-LLM," arXiv:2506.20415, 2025.
[6] Assertain, arXiv:2604.01583, 2025.
[7] K. Hui et al., "TrojanForge," arXiv:2405.15184, 2024.
[8] J. Bhandari et al., "SENTAUR," arXiv:2407.12352, 2024.
[9] M. O. Faruque et al., "GHOST," Electronics, 2025.
[10] A. Moschos et al., "0ena," ACNS, 2024.
[11] S. Deb et al., "SoC-HTs," IEEE, 2023.
[12] Trust-Hub, "Hardware Trojan Benchmarks," trust-hub.org.
[13] HT-PGFV, Electronics, 2024.
[14] J. Wu and D. Forte, "QuEST," ITC, 2025.

---

## Ethical Use

This tool is intended for security research and academic evaluation of hardware
Trojan detection methods only. It must not be used to insert Trojans into
production hardware or compromise real systems.

---

## Acknowledgments

Built on the **lowRISC Ibex** RISC-V processor (Apache 2.0):
https://github.com/lowRISC/ibex

Trojan taxonomy based on **Trust-Hub** Hardware Trojan benchmarks:
https://trust-hub.org

Funded by the Estonian Research Council grant **PSG837**.

---

## Citation

```bibtex
@article{imtiaz2025autoassert,
  title   = {AutoAssert-RV: Automated Security Assertion Translation
             and Formal Trojan Evaluation for RISC-V Processors},
  author  = {Imtiaz, Sharjeel and Reinsalu, Uljana and Ghasempouri, Tara},
  journal = {IEEE Access},
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

**Version:** 1.1.0 | **Last Updated:** April 2026
