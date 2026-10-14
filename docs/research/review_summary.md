# Novelty Review — Summary (14 October 2026)

Four research agents reviewed the three advisor-approved metrics:

- **Red team** ([red_team.md](red_team.md)) attacked the novelty claims.
- **Blue team** ([blue_team.md](blue_team.md)) built the strongest honest defence.
- **Literature survey** ([literature_survey.md](literature_survey.md)) collected 72 papers. Most summaries come from abstracts, and about 15 entries are marked unverified.
- **Metric scout** ([new_metric_ideas.md](new_metric_ideas.md)) looked for a possible new metric.

Every citation comes from a search result the agents actually found. Check all BibTeX entries before submission.

## Verdicts

| Item | Red team | Blue team | Agreed position |
|---|---|---|---|
| SRTLC | Partly overlapping and weak. COI coverage is standard, and the tool vendors themselves say it overestimates coverage; proof-core coverage is the accepted fix. | Defensible as coverage of expert-defined security logic. | Keep it. Justify the COI choice, or switch to proof-core coverage. Only the NS31A-based denominator is new. |
| AER | The mechanism is pre-empted. The XOR free-input model with JasperGold appears in the Chuah et al. RISC-V Summit Europe 2025 abstract. | Could not verify that attribution. Sees AER as a form of formal mutation coverage (Chockler et al., CAV 2001). | Keep it, but define property P separately from the assertion, and make AER graded rather than 0/1. Credit the prior models. |
| SAPC | Pre-empted by our co-author's work (Eslami, Ghasempouri, Pagliarini, ISQED 2022; SCARF 2024). As defined it may also be tautological, since a cone of influence already contains every path into the signal. | Same prior work. Fix: count SPV functional paths from input, through the security signal, to the assertion's failure output. | Redefine SAPC as functional paths. Present it as a security-scoped extension of the group's 2022 metric, which was never tested against Trojans. |
| AQS | Partly overlapping with existing assertion ranking (Hertz et al. ASP-DAC 2019; Ayalasomayajula HOST 2024). | The novel part is the combination of FPV evasion and SPV path evidence in one score. | Claim only the combination. Avoid "first". |
| "AQS predicts detection" | Looks novel but is fragile: small sample, mostly 0/1 detection counts, Trojans written by the same group, refinement re-tested on the same Trojans. | This is the strongest claim, if it is done properly. | Use Spearman with confidence intervals, compare against baselines, and test refinement on held-out Trojans. |

## Facts that change the paper

1. **arXiv 2201.01130 is our group's own work** (Tara Ghasempouri is a co-author). Present SAPC as an extension of it.
2. **Our ISCAS 2025 paper** (Imtiaz, Reinsalu, Ghasempouri, arXiv 2502.10194) already uses TDER and TPI on Ibex. It reports 5 security modules and 100% detection. Cite it and reuse those metrics rather than claim them. The BEC 2026 AutoTrans paper is the source of the 68 assertions across 9 modules; confirm how the two papers relate.
3. **"Chuah ISQED 2026" exists**: "Reliable Hardware Trojan Detection for RISC-V Processors using Formal Verification and Automation". Its abstract does not mention XOR or JasperGold. "Formal Verification of Security Properties on RISC-V Processors" is MBMV 2024, not MEMOCODE 2023.
4. **TrustHub's own designs are contested** (Krieg, ICCAD 2023). Write "TrustHub categories, new Ibex RTL".
5. **Do not claim the first RISC-V Trojan benchmark.** Damljanovic (DDECS 2021) and RISC-V-Dilemma-1 (2025) already exist.
6. **"AutoAssert" also collides with arXiv 2508.07371.** Renaming is needed.

## Proposed novelty statement (blue team, hedged)

> AQS is a per-assertion security quality score computed by formal analysis on Trojan-free RTL. It combines coverage of expert-defined security signals (SRTLC), FPV-based evasion resistance (AER) and SPV-based functional-path coverage (SAPC). On 68 proven Ibex assertions across 9 security modules, AQS computed before any Trojan exists correlates with detection on an independent 39-Trojan benchmark, and AQS-guided refinement improves detection on held-out Trojans.

Write "to our knowledge, the first to combine FPV evasion and SPV path evidence in one per-assertion score". Say "correlates with", never "proves".

## Strengthening experiments

1. Spearman or Kendall correlation with bootstrap confidence intervals. Control for module and cone size, and cluster DO/ETI/CF/MT because they share one RTL file. Add a top-quartile vs bottom-quartile precision figure.
2. Baselines: plain COI coverage, proof-core coverage, Eslami node coverage, mutation kill rate, signal count, and a random ranking.
3. Refinement tested on held-out Trojans. Add a control that adds the same number of template assertions on random signals.
4. Ablation: drop each metric in turn and check ranking stability under random weights.
5. Head-to-head comparisons on the same 68 assertions: AER vs mutation kill rate, and SAPC vs Eslami coverage. Include one worked trace showing a gap the older metric misses.

## New metric scout

Stay at 3 metrics. Turn AER into a graded **Evasion Budget**: the cheapest tampering (bits × cycles) that evades the assertion, normalised to [0,1]. It needs no extra JasperGold runs and gives AQS the spread that the correlation needs. Privilege/reset context coverage is a possible 4th metric, but only if a CSR and debug pilot shows its scores vary between assertions.

## Decisions needed (Sharjeel and advisor)

1. Redefine SAPC as SPV functional paths to the assertion output: yes or no.
2. Define AER's reference property P (NS31A property or golden check), and choose graded or binary AER.
3. Choose Pearson or Spearman, and whether to hold out Trojans for the refinement test.
4. Settle which paper is the assertion source (BEC 2026 AutoTrans vs ISCAS 2025) and how to cite both.
