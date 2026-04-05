# AutoAssert-RV

**Automated Security Assertion Translation and Formal Hardware Trojan Evaluation for RISC-V Processors**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![Status: Research](https://img.shields.io/badge/status-research-orange.svg)]()

---

## Overview

**AutoAssert-RV** is a Claude-assisted pipeline for automated security assertion translation
and formal hardware Trojan evaluation for RISC-V processors. It enables security researchers to:

- **Translate assertions automatically** — Map security properties from a reference processor to a target processor using LLM assistance
- **Evaluate formal detection** — Run JasperGold formal verification against 39 hardware Trojans
- **Measure detection quality** — Compute 4 novel security metrics (TAR, WTDR, AAD, AER)
- **Reproduce paper results** — Full reproducible experiment suite with 63 JasperGold runs

AutoAssert-RV operates in three phases. Phase 1 covers RTL parsing and signal extraction.
Phase 2 covers Claude-assisted assertion translation with compiler validation.
Phase 3 covers JasperGold formal verification (baseline, trojaned, and AER runs).

This repository accompanies the paper:
> **"AutoAssert-RV: Automated Security Assertion Translation and Formal Trojan Evaluation for RISC-V Processors"**
> S. Imtiaz, U. Reinsalu, T. Ghasempouri
> Tallinn University of Technology — Grant PSG837

---

## Key Features

### Claude-Assisted Translation Pipeline
Automated signal mapping from NS31A (reference) to Ibex (target) RISC-V processor,
with compiler-in-the-loop validation (QuestaSim, max 3 retries per module).

### RV-Trojan Benchmark
39 trojaned RTL files covering 9 security-critical Ibex modules across
6 TrustHub attack categories:

| # | Category | Target | Mechanism |
|---|----------|--------|-----------|
| 1 | Availability | Control enable signals | Periodic duty-cycle stall |
| 2 | Covert | Single-bit output | Timing channel encoding |
| 3 | Denial of Service | Control enable signals | Permanent path disable |
| 4 | Integrity | Data output signals | XOR data corruption |
| 5 | Leakage | Stable output ports | Secret data routing |
| 6 | Privilege | Privilege-mode registers | Forced M-mode escalation |

### Four Novel Security Metrics

| Metric | Formula | Description |
|--------|---------|-------------|
| TAR | correct auto-mappings / total × 100% | Translation automation quality |
| WTDR | Σ(TPI×detected) / ΣTPI × 100% | Stealth-weighted detection rate |
| AAD | triggered assertions / module total × 100% | Per-Trojan detection breadth |
| AER | JasperGold SPV Proven/CEX | Formal evasion resistance |

### 63 JasperGold Formal Verification Runs
9 modules × 7 files each (1 original + 6 trojaned).
ibex_controller covers 4 logical modules (DO, ETI, CF, MT) with shared DUT.

---

## Module Coverage

| Module ID | RTL File | Security Concern |
|-----------|----------|-----------------|
| PMP | `ibex_pmp.sv` | Physical memory protection |
| CSR | `ibex_cs_registers.sv` | Control/status registers |
| DO | `ibex_controller.sv` | Decode/output control |
| ETI | `ibex_controller.sv` | Exception/trap interface |
| CF | `ibex_controller.sv` | Control flow |
| MT | `ibex_controller.sv` | Mode transitions |
| MA | `ibex_load_store_unit.sv` | Memory access |
| IE | `ibex_id_stage.sv` + `ibex_ex_block.sv` | Instruction execution |
| RU | `ibex_wb_stage.sv` | Register writeback |

---

## Repository Structure

```
autoassert-rv/
|
|-- pipeline/                   Phase 1-2: RTL parsing and assertion translation
|   |-- prompts/                Claude prompt templates (one per module)
|   |-- signals/                Extracted signal JSON files (gitignored)
|   |-- parse_rtl.py            RTL parser — outputs MODULE_signals.json
|   |-- translate.py            Claude-assisted assertion translator
|   |-- validate.py             QuestaSim compiler validator (3-retry loop)
|   |-- run_pipeline.sh         Single-module pipeline runner
|   `-- batch_all.sh            All 9 modules in sequence
|
|-- rtl/
|   |-- original/               Clean Ibex RTL (9 modules)
|   `-- trojaned/               39 trojaned RTL files (6 per module)
|
|-- assertions/
|   |-- source/                 NS31A reference assertions (SVA)
|   `-- translated/             Ibex translated bind files (SVA)
|
|-- jasper/                     Phase 3: Formal verification
|   |-- constraints/            JasperGold constraint files
|   |-- oracles/                Oracle/golden assertion modules
|   |-- results/                JasperGold output (gitignored)
|   |-- fpv_run.tcl             FPV baseline TCL script
|   |-- spv_aer.tcl             SPV AER TCL script
|   `-- run_all.sh              Batch all 63 runs
|
|-- metrics/
|   |-- outputs/                Computed metric tables (gitignored)
|   |-- compute_tpi.py          TPI computation
|   |-- build_matrix.py         Detection matrix builder
|   |-- compute_metrics.py      TAR, WTDR, AAD, AER computation
|   `-- plot_tpi_aad.py         TPI-AAD correlation plot
|
`-- docs/
    |-- paper/                  LaTeX source
    `-- figures/                Paper figures
```

---

## Requirements

- Python 3.9+ with pyverilog (`pip install pyverilog`)
- Claude Code CLI (installed and authenticated)
- QuestaSim (for compile validation in Phase 2)
- JasperGold (for formal verification in Phase 3, licence required)
- Ibex RISC-V RTL (place in `rtl/original/`)

---

## Quick Start

```bash
# 1. Clone
git clone https://github.com/Sharjeelimtiaz27/autoassert-rv
cd autoassert-rv

# 2. Parse RTL signals for one module
python pipeline/parse_rtl.py --input rtl/original/ibex_pmp.sv

# 3. Run translation pipeline for one module
bash pipeline/run_pipeline.sh pmp

# 4. Run translation for all 9 modules
bash pipeline/batch_all.sh

# 5. Run JasperGold experiments (all 63 runs)
bash jasper/run_all.sh

# 6. Compute all metrics
python metrics/compute_tpi.py
python metrics/build_matrix.py
python metrics/compute_metrics.py
python metrics/plot_tpi_aad.py
```

---

## JasperGold Commands

```bash
# FPV baseline (clean RTL)
MODULE=ibex_pmp BIND=ibex_pmp_bind jg -tcl jasper/fpv_run.tcl

# FPV trojaned RTL
MODULE=ibex_pmp BIND=ibex_pmp_bind DUT=ibex_pmp_trojan_DoS jg -tcl jasper/fpv_run.tcl

# AER (evasion resistance)
MODULE=ibex_pmp PROP=PMP_1 jg -tcl jasper/spv_aer.tcl

# Batch all 63 runs
bash jasper/run_all.sh
```

---

## Comparison with Related Work

### Assertion translation and LLM-based generation

| Feature | AutoAssert-RV (Ours) | Transys [1] | Chuah et al. [2] | Kande et al. [3] | SV-LLM [4] |
|---------|---------------------|-------------|------------------|-----------------|-------------|
| Task | Cross-arch translation | Rule-based translation | Manual assertion writing | LLM generation | Multi-agent verification |
| Input | Validated assertions + RTL | RTL + constraints | RISC-V spec | RTL + NL comments | SoC design files |
| Reuses validated assertions | Yes | Yes | N/A | No | No |
| Target architecture | RISC-V Ibex | Generic | RISC-V NS31A | ISCAS benchmarks | Generic SoC |
| Formal backend | JasperGold FPV + SPV | None | JasperGold FPV | Simulation only | Simulation only |
| Trojan evaluation | Yes — 39 real Trojans | None | None | None | None |
| Novel metrics | TAR, WTDR, AAD, AER | None | None | None | None |
| Open source | Yes (MIT) | Partial | No | No | No |

### Formal Trojan detection

| Feature | AutoAssert-RV (Ours) | HT-PGFV [5] | QuEST [6] |
|---------|---------------------|-------------|-----------|
| Detection method | SVA bind files + JasperGold FPV | Auto-generated SVA + formal | Shannon entropy metrics |
| Target | RISC-V processor modules | ISCAS / Trust-Hub | Generic IP |
| Trojan source | TrustHub taxonomy — 39 files | Trust-Hub | Trust-Hub |
| Trojan categories | 6 (full TrustHub set) | Mixed | Leakage focus |
| Novel metrics | WTDR, AAD, AER | None | Entropy-based |
| RISC-V specific | Yes | No | No |

**References:**
[1] R. Zhang and C. Sturton, "Transys," IEEE S&P, 2020.
[2] C. S. Chuah et al., "Formal Verification of Security Properties on RISC-V Processors," MEMOCODE, 2023.
[3] R. Kande et al., "(Security) Assertions by Large Language Models," IEEE TIFS, 2024.
[4] D. Saha et al., "SV-LLM: An Agentic Approach for SoC Security Verification," arXiv:2506.20415, 2025.
[5] HT-PGFV, Electronics, 2024.
[6] J. Wu and D. Forte, "QuEST," ITC, 2025.

---

## Ethical Use

This tool is intended for security research, formal verification testing, and academic
evaluation of hardware Trojan detection methods. It must not be used to insert Trojans
into production hardware or compromise real systems. Users bear full responsibility for
ensuring ethical and lawful use.

---

## Acknowledgments

This work targets the **lowRISC Ibex** RISC-V processor (Apache 2.0):
https://github.com/lowRISC/ibex

Trojan taxonomy based on the **Trust-Hub** Hardware Trojan benchmark suite:
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
- **Institution:** Tallinn University of Technology (TalTech), Estonia
- **Email:** sharjeel.imtiaz@taltech.ee
- **Repository:** https://github.com/Sharjeelimtiaz27/autoassert-rv
- **Issues:** https://github.com/Sharjeelimtiaz27/autoassert-rv/issues

---

**Version:** 1.0.0 | **Last Updated:** 5 April 2026
