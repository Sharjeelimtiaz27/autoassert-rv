# AutoAssert-RV (working name)

**Formal Security Assertion Quality Ranking for RISC-V Processors**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Status: Research](https://img.shields.io/badge/status-research-orange.svg)]()

> Final system name pending advisor decision (a DATE 2026 paper already uses
> "AutoAssert"). Candidates: AQS-RV, SecRank-RV, FormalRank-RV.
> Target venue: IEEE European Test Symposium (ETS) 2027.

---

## What this repository does

Given a set of security assertions that are already formally proven on a
clean RISC-V design, this work asks: **which assertions are actually strong,
and which have gaps?** It scores every assertion with three formal metrics,
computed with Cadence JasperGold on clean RTL before any Trojan is used, and
then checks that the score predicts real Trojan detection.

**Input:** 68 security assertions for the Ibex RISC-V core, covering 9
security modules. They come from our earlier paper *AutoTrans: AI-Assisted
Automatic Translation of Security Assertions for RISC-V Processors*
(BEC 2026), where they were translated from NS31A expert properties and
proven non-vacuous with JasperGold. This repository does no translation.

## The three metrics

| Metric | Full name | Engine | Question it answers |
|---|---|---|---|
| SRTLC | Security RTL Coverage | JasperGold cone-of-influence (COI) | How much expert-defined security logic does the assertion set watch? |
| AER | Assertion Evasion Resistance | JasperGold FPV + free test signal | Can any tampering of a security signal break the property while the assertion stays silent? |
| SAPC | Security Assertion Path Coverage | JasperGold SPV | Are there structural input paths into security signals that no assertion watches? |

**Assertion Quality Score:** `AQS = (SRTLC_contrib + AER + SAPC_contrib) / 3`
gives one ranked score per assertion.

Two descriptive **comparison tables** (not metrics) sit beside the score:
each assertion mapped to its CWE weakness (with CVSS severity, citing
NIST CSWP 45), and each assertion mapped to the TrustHub Trojan categories
it can address.

Full step-by-step explanation: [docs/metrics_explained.md](docs/metrics_explained.md).

## Five stages

| # | Stage | JasperGold runs | Uses Trojans? |
|---|---|---|---|
| 1 | Assertion Validation Stage (AVS) | 0 new (done in BEC 2026 paper) | No |
| 2 | Assertion Quality Ranking Stage (AQRS) | 20 (10 AER FPV + 10 SAPC SPV) | No |
| 3 | Trojan Evaluation Stage (TES) | 39 FPV | Yes |
| 4 | Assertion Refinement Stage (ARS) | 0 (Python templates) | No |
| 5 | Refined Assertion Validation Stage (RAVS) | 39 FPV | Yes |

Stage 3 also tests the ranker: AQS is correlated with how many Trojans each
assertion detects.

## Trojan benchmark

39 trojaned RTL files for the 7 Ibex RTL files behind the 9 security modules
(`rtl/ibex/trojaned_rtl/`), built from the TrustHub taxonomy independently
of the assertion set.

| Category | Mechanism |
|---|---|
| Availability | Periodic duty-cycle stall on control signals |
| Covert | Timing channel encoding on a single-bit output |
| Denial of Service | Permanent path disable on control enables |
| Integrity | XOR data corruption on output signals |
| Leak | Secret data routed to stable output ports |
| Privilege | Forced machine-mode escalation on privilege registers |

PMP has no Availability Trojan; the load-store unit and write-back stage
have no Privilege Trojan.

## Repository layout

```
assertion_dataset/     NS31A source properties by category (CSV)
assertions/
  source_sva/          SVA from the BEC 2026 paper (input)
  translated/          verified bind files used in every stage
docs/
  metrics_explained.md step-by-step explanation of the metrics
  metrics_reference.md literature notes and novelty checks
  research/            red-team, blue-team and literature reports
  paper/               related papers (PDF)
jasper/
  oracles/             AER test-signal wrappers (one per bind file)
results/step1..step5/  JasperGold output per stage
results/metrics/       computed metric tables
rtl/ibex/
  original/            unmodified Ibex RTL
  trojaned_rtl/        39 benchmark Trojans (7 RTL files)
  generated_trojans/   Trojan snippets used to build the benchmark
```

## Requirements

- Python 3.10+ with pandas
- Cadence JasperGold (FPV and SPV apps)

## Ethical use

For security research and academic evaluation only. Do not use it to insert
Trojans into production hardware.

## Acknowledgments

Built on the [lowRISC Ibex](https://github.com/lowRISC/ibex) core (Apache 2.0).
Trojan taxonomy from [TrustHub](https://trust-hub.org).
Funded by the Estonian Research Council grant PSG837.

## License

MIT — see [LICENSE](LICENSE).

## Contact

Sharjeel Imtiaz, Tallinn University of Technology (TalTech) —
sharjeel.imtiaz@taltech.ee
