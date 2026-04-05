# AutoAssert-RV

Automated security assertion translation and formal hardware Trojan
evaluation for RISC-V processors, using LLM-assisted RTL analysis
and JasperGold formal verification.

This repository accompanies the paper:
> **"AutoAssert-RV: Automated Security Assertion Translation and
> Formal Trojan Evaluation for RISC-V Processors"**
> S. Imtiaz, U. Reinsalu, T. Ghasempouri
> Tallinn University of Technology — PSG837

---

## What this does

Takes security assertions from a reference RISC-V processor (NS31A),
automatically translates them to a target processor (Ibex) using a
Claude-assisted pipeline, then formally verifies them against 39
hardware Trojans from the TrustHub taxonomy using JasperGold.

**Key results:**
- 9 security-critical modules covered (PMP, CSR, DO, ETI, CF, MA, IE, RU, MT)
- 39 Trojans across 6 TrustHub attack categories
- 63 JasperGold formal verification runs
- 4 novel security metrics: TAR, WTDR, AAD, AER

---

## Requirements

- Python 3.9+ with pyverilog (`pip install pyverilog`)
- Claude Code CLI (installed and authenticated)
- QuestaSim (for compile validation)
- JasperGold (for formal verification, licence required)
- Ibex RISC-V RTL (included in rtl/)

---

## Quick start

```bash
# 1. Clone
git clone https://github.com/Sharjeelimtiaz27/autoassert-rv
cd autoassert-rv

# 2. Run translation pipeline for one module
bash pipeline/run_pipeline.sh pmp

# 3. Run translation for all 9 modules
bash pipeline/batch_all.sh

# 4. Run JasperGold experiments (all 63 runs)
bash jasper/run_all.sh

# 5. Compute all metrics
python metrics/compute_tpi.py
python metrics/build_matrix.py
python metrics/compute_metrics.py
python metrics/plot_tpi_aad.py
```

---

## Repository structure

```
autoassert-rv/
├── pipeline/      # Claude-assisted translation tool
├── rtl/           # Ibex original and trojaned RTL files
├── assertions/    # Source (NS31A) and translated (Ibex) SVA bind files
├── jasper/        # JasperGold TCL scripts and oracle modules
├── metrics/       # Metric computation and plotting scripts
└── docs/          # Paper LaTeX source
```

---

## Metric definitions

| Metric | Formula | Description |
|--------|---------|-------------|
| TAR | correct auto-mappings / total × 100% | Translation automation quality |
| WTDR | Σ(TPI×detected) / ΣTPI × 100% | Stealth-weighted detection rate |
| AAD | triggered assertions / module total × 100% | Per-Trojan detection breadth |
| AER | JasperGold SPV Proven/CEX | Formal evasion resistance |

---

## Citation

```bibtex
@article{imtiaz2025autoassert,
  title   = {AutoAssert-RV: Automated Security Assertion Translation
             and Formal Trojan Evaluation for RISC-V Processors},
  author  = {Imtiaz, Sharjeel and Reinsalu, Uljana
             and Ghasempouri, Tara},
  journal = {IEEE Access},
  year    = {2025}
}
```

---

## License
MIT — see LICENSE file.
