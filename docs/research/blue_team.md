# Blue Team Report: Novelty and Positioning for the AQS Ranker Paper (ETS 2027)

Date: 2026-10-14. Scope: build the strongest *honest* novelty case for the assertion-quality ranker (SRTLC, AER, SAPC, AQS) and the Trojan-based validation. Every citation below came from a web search or fetch in this session. Where I could not open the full text, that is stated.

---

## 0. Findings that change the positioning

Read these first. Several of them are risks that a reviewer from the TalTech / formal-HT community would raise immediately.

1. **A per-assertion, SPV-based, Trojan-free security score already exists, and our co-author wrote it.** Eslami, **Ghasempouri**, Pagliarini, ISQED 2022 (arXiv 2201.01130), define *Security Coverage = reachable nodes / total nodes* **per assertion**. They compute it with JasperGold SPV over (netlist node, assertion output) pairs, use it to **rank and select** assertions, and do not need any Trojan ("does not rely on the HT activation mechanism"). I extracted this from the PDF text, Eq. (1) and Sec. II-C. SCARF (IEEE TC 2024, arXiv 2402.12162) extends it. **So we cannot claim "first per-assertion formal security score", "first SPV-based assertion metric", or "first Trojan-free assertion ranking".** The defensible framing is that SAPC is a *security-scoped refinement of our group's own SC metric*. That makes the self-citation a strength, because it gives the paper a clear lineage.
   - SC's denominator is *all synthesized netlist nodes*. Its reported values are very low (on average 6.8% of nodes for B19-T500; at most 4.77% on OpenTitan reg-tops), so it penalizes small, focused security checks. This is the concrete weakness SAPC fixes.
   - SC was demonstrated on one TrustHub circuit (B19-T500) and OpenTitan register tops, with an overhead trade-off. There is **no correlation study of SC against detection counts over a Trojan benchmark**.
2. **TDER and TPI are already defined in our own ISCAS 2025 paper** (Imtiaz, Reinsalu, Ghasempouri, arXiv 2502.10194). That paper already evaluated translated assertions on Ibex against LLM-generated Trojans. The validation therefore cannot be sold as "the first Trojan evaluation of security assertions on Ibex". TDER and TPI must be cited as reused from it. WTDR and AAD should be presented as reporting metrics, not as contributions.
3. **Iman, Ahmadi-Pour, Drechsler, Ghasempouri, NorCAS 2024** (same group) mined RISC-V security assertions and reported per-assertion Trojan detection ("ratio of security assertions detecting Trojans", 6.3% vs HARM's 2.4%). This is another in-group precedent for counting how many Trojans each assertion detects.
4. **AER is, formally, a mutation- or fault-injection-based coverage check.** The classic definition is "a component is covered if changing it falsifies the specification" (Chockler, Kupferman, Vardi, TACAS/CAV 2001). Formal fault injection with assertions as detectors also exists: Bingham and Lach's ABVFI, and Zuccalà et al. MEMOCODE 2024. The Jasper COV App itself ships COI, proof-core and **mutation**-based checker coverage. AER must be positioned as a *security-oracle-specific, per-assertion instance* of formal mutation coverage, not as a new paradigm.
5. **Chuah et al.**: DBLP confirms "Formal Verification of Security Properties on RISC-V Processors", MEMOCODE 2023, pp. 159–168. The ISQED 2026 abstract confirms model-checking HT detection on RISC-V with signal-connection properties and a complete property set for PC-modifying Trojans. **I could not verify the "free XOR variable" HT model in either paper**, because no full text was reachable. Do not attribute the XOR mechanism to Chuah until someone has read the PDF.
6. **TrustHub validity is contested.** Krieg, "Reflections on Trusting TrustHUB", ICCAD 2023, finds that only 3 of 83 TrustHub designs are real Trojans. Our benchmark only *borrows TrustHub categories* and is newly written for Ibex. Say exactly that, and pre-empt the critique.

---

## 1. Literature search log (25 searches/fetches; key hits)

| # | Query theme | Key verified result |
|---|---|---|
| 1 | SPV security coverage for assertions | Eslami/Ghasempouri/Pagliarini ISQED 2022 — https://arxiv.org/abs/2201.01130 (PDF text read: Eq. 1, SPV flow, assertion ranking) |
| 2 | SCARF | IEEE TC 73(12) 2024 — https://arxiv.org/abs/2402.12162 |
| 3 | Information-flow coverage | Meza & Kastner, hyperflow graphs (simulation IFT, OpenTitan, no Trojans, no assertion scoring) — https://arxiv.org/abs/2304.08263 |
| 4 | Mutation assertion quality | Krishnamurthy et al. 2026, mutation-guided refinement of LLM RTL assertions, not security — https://arxiv.org/abs/2606.21451 |
| 5 | GateTruth | Bhadra 2026, mutation audit of testbenches (not assertions, not security) — https://arxiv.org/abs/2608.12635 |
| 6 | Chuah | ISQED 2026 abstract — https://isqed.org/English/Archives/2026/Technical_Sessions/79.html ; MEMOCODE 2023 entry via DBLP — https://dblp.org/pid/61/8263 (full text not read) |
| 7 | NIST CSWP 45 | https://csrc.nist.gov/pubs/cswp/45/metrics-and-methodology-for-hardware-security-cons/final |
| 8 | LAsset | DATE 2026 — https://arxiv.org/abs/2601.02624 |
| 9 | Assertion-set completeness & HT | Fern & Cheng, DATE 2019 (simulation traces + assertion set) — https://past.date-conference.com/proceedings-archive/2019/html/0322.html |
| 10 | Formal coverage theory | Chockler, Kupferman, Vardi, coverage metrics for temporal-logic model checking — https://www.cs.huji.ac.il/~ornak/publications/tacas01.pdf |
| 11 | COI vs proof-core | Jasper COV App (COI, proof core, mutation) — https://newstaging.cadence.com/ko_KR/home/tools/system-design-and-verification/formal-and-static-verification/jasper-gold-verification-platform/jaspergold-design-coverage-verification-app.html ; AssertionForge uses Jasper COI coverage per assertion — https://arxiv.org/abs/2503.19174 |
| 12 | Assert-O | Ranks LLM-generated security properties by associated vulnerability (OpenTitan) — https://files-prod.tii.ae/2025-11/Assert-O-%20Context-based%20Assertion%20Optimization%20using%20LLMs.pdf |
| 13 | RISC-V assertion mining + HT | Iman et al. NorCAS 2024 (PDF text read) — https://agra.informatik.uni-bremen.de/doc/konf/norcas2024_sap.pdf |
| 14 | Own prior work | ISCAS 2025 — https://arxiv.org/abs/2502.10194 ; AutoTrans BEC 2026 — https://arxiv.org/abs/2609.10057 |
| 15 | HT-PGFV | Qin et al., Electronics 13(21):4286, 2024 — https://www.mdpi.com/2079-9292/13/21/4286/htm |
| 16 | TrustHub validity | Krieg, ICCAD 2023 — https://repositum.tuwien.at/handle/20.500.12708/191214 |
| 17 | Trojan generators | TrojanForge — https://arxiv.org/html/2405.15184v3 ; SENTAUR — https://arxiv.org/abs/2407.12352 |
| 18 | Formal fault injection with assertions | Zuccalà et al. (ESWEEK/MEMOCODE 2024 proceedings folder) — https://web.eng.fiu.edu/gaquan/Papers/ESWEEK24Papers/CPS-Proceedings/pdfs/MEMOCODE/780200a077/780200a077.pdf ; Bingham & Lach ABVFI — https://webhost.laas.fr/TSF/WDSN09/WDSN09_files/Texts/WDSN09-2-4-Bingham.pdf |
| 19 | Coverage vs effectiveness (method model) | Inozemtseva & Holmes ICSE 2014 — https://cs.ubc.ca/~rtholmes/papers/icse_2014_inozemtseva.pdf |
| 20 | Mutants as proxy for real faults | Just et al. FSE 2014 — https://people.cs.umass.edu/~rjust/publ/mutants_real_faults_fse_2014.pdf |
| 21 | Security-property completeness metrics | DATE 2012 "guiding coverage metric" for property checking (JKU) — https://ics.jku.at/files/2012date_gcm.pdf (only a search snippet was seen; check the exact title and authors before citing) ; MBMV 2025 coverage metrics for security property verification — https://www.vde-verlag.de/proceedings-en/456515019.html (abstract not read) |
| 22 | System-level IF security coverage | Coşkun et al. ASP-DAC 2024 (SiMiT, VP-based) — https://www.aspdac.com/aspdac2024/archive/pdf/4A-1.pdf |
| 23 | LLM security assertions | Kande et al., (Security) Assertions by LLMs — https://arxiv.org/abs/2306.14027 |
| 24 | Hardware security coverage | Kastner et al. VTS 2021 slides (property-driven security, threat model) — https://kastner.ucsd.edu/wp-content/uploads/2021/06/admin/vts21-hwsec-coverage.pdf |
| 25 | Unspecified-functionality HTs | Fern & Cheng ICCAD 2015 (citation confirmed only via CV listing; no full text) |

**Conclusion of the search.** I found **no** prior work that satisfies all three of the following at once:

- (a) a per-assertion security quality score computed formally on Trojan-free RTL;
- (b) that score combines *behavioral* evidence (FPV with a free perturbation) with *structural/functional-path* evidence (SPV);
- (c) the score is tested for *predictive validity*, as a statistical correlation with detection counts, on a multi-module Trojan benchmark of a real RISC-V core.

Properties (a) and SPV-structural evidence on their own are covered by Eslami 2022 and SCARF. Behavioral mutation coverage on its own is classic (Chockler 2001, Jasper COV mutation). A Trojan evaluation of assertions on RISC-V on its own exists in our ISCAS 2025 paper and Iman NorCAS 2024. **The novelty is the combination and, above all, the predictive-validation question.**

---

## 2. Differentiation table

Legend: Y = yes, N = no, P = partial, ? = could not verify from the accessible text.

| Prior work | Per-assertion score | Formal (not sim) | Behavioral evasion check | Structural/path check | Computed Trojan-free | Validated vs Trojans | RISC-V processor | Security-specific denominator |
|---|---|---|---|---|---|---|---|---|
| Eslami et al. ISQED 2022 — [2201.01130](https://arxiv.org/abs/2201.01130) | **Y** (SC per assertion, ranked) | Y (SPV) | N | **Y** (SPV node→assertion paths) | **Y** | P (B19-T500 demo; no correlation) | N (OpenTitan reg-tops are peripherals; Viper example in text) | **N** (all netlist nodes) |
| SCARF, IEEE TC 2024 — [2402.12162](https://arxiv.org/abs/2402.12162) | Y | Y (SPV) | N | Y | Y | P (overhead/SC; no detection correlation seen in abstract) | N | N |
| Meza & Kastner 2023 — [2304.08263](https://arxiv.org/abs/2304.08263) | N (design flows) | N (sim IFT; formal optional) | N | Y (graph paths) | Y | N | N (OpenTitan OTP) | P (asset-based) |
| Fern & Cheng DATE 2019 — [link](https://past.date-conference.com/proceedings-archive/2019/html/0322.html) | N (set-level completeness) | N (sim traces) | P | N | Y | P (FIFO, AXI4-Lite leakage Trojans) | N | N |
| Chockler/Kupferman/Vardi 2001 — [tacas01](https://www.cs.huji.ac.il/~ornak/publications/tacas01.pdf) | P (per-spec coverage) | Y | Y (mutation falsifies spec) | N | Y | N | N | N |
| Jasper COV App (COI / proof core / mutation) — [Cadence](https://newstaging.cadence.com/ko_KR/home/tools/system-design-and-verification/formal-and-static-verification/jasper-gold-verification-platform/jaspergold-design-coverage-verification-app.html) | Y | Y | P (mutation) | P (COI) | Y | N | N | N |
| Krishnamurthy et al. 2026 — [2606.21451](https://arxiv.org/abs/2606.21451) | Y (reject weak assertions) | Y (formal + mutation) | Y (mutation kill) | N | Y | N | N | N |
| Chuah et al. ISQED 2026 — [ISQED](https://isqed.org/English/Archives/2026/Technical_Sessions/79.html) | N (property-set completeness for PC Trojans) | Y | ? (free-variable HT model not verified) | P (signal-connection properties) | Y | Y | **Y** | N (PC-focused) |
| Imtiaz et al. ISCAS 2025 (own) — [2502.10194](https://arxiv.org/abs/2502.10194) | N (set-level TDER) | P | N | N | N | Y | **Y (Ibex)** | N |
| Iman et al. NorCAS 2024 — [PDF](https://agra.informatik.uni-bremen.de/doc/konf/norcas2024_sap.pdf) | P (counts assertions detecting each HT) | N (mined from sim) | N | N | N | Y (3 HTs) | Y | N |
| HT-PGFV, Electronics 2024 — [MDPI](https://www.mdpi.com/2079-9292/13/21/4286/htm) | N (generates HT assertions) | Y (IF analysis) | N | P (Trojan path tracing) | N (uses Trojan-infected signals) | Y | ? | N |
| Assert-O 2025 — [PDF](https://files-prod.tii.ae/2025-11/Assert-O-%20Context-based%20Assertion%20Optimization%20using%20LLMs.pdf) | Y (ranked by vulnerability) | N (LLM) | N | N | Y | N | N (OpenTitan) | P (vulnerability-based) |
| Zuccalà et al. 2024 formal FI — [PDF](https://web.eng.fiu.edu/gaquan/Papers/ESWEEK24Papers/CPS-Proceedings/pdfs/MEMOCODE/780200a077/780200a077.pdf) | P | Y | Y (bit-flip faults vs assertions) | N | Y | N (safety faults) | N | N |
| **This work (AQS)** | **Y** | **Y** | **Y (AER)** | **Y (SAPC, SPV)** | **Y** | **Y (39 HTs, correlation)** | **Y (Ibex, 9 modules)** | **Y (NS31A-derived signals)** |

The one row that is "Y" across the board is ours. That holds only if SAPC really uses SPV *functional* paths and not plain syntactic COI (see §4), and only if AER has a security oracle that is independent of the assertion under test.

---

## 3. Novelty wording

### 3.1 Abstract novelty sentence (recommended)

> "Security assertions that are proven and non-vacuous can still differ widely in how well they would expose a hardware Trojan, yet there is no established way to rank them before any Trojan is known. We present AQS, a per-assertion security quality score computed by formal analysis on Trojan-free RTL. AQS combines (i) Security RTL Coverage over expert-defined security signals, (ii) Assertion Evasion Resistance, an FPV check of whether an unconstrained perturbation of a security signal can break the intended security property without firing the assertion, and (iii) Security Assertion Path Coverage, which uses JasperGold SPV to find functional input paths to security signals that no assertion observes. On 68 proven assertions covering 9 security modules of the Ibex RISC-V core, AQS computed before any Trojan is inserted is positively correlated with each assertion's detection count on an independent 39-Trojan benchmark (Spearman ρ = X, 95% CI [a, b]), and outperforms coverage-only baselines [*only if experiment E2 shows it*]. Refinement guided by AQS gaps raises Trojan detection from X% to Y% on held-out Trojans."

Hedging rules for the abstract:
- Do not use "first" anywhere in the abstract.
- Do not claim the score "proves" or "guarantees" detection. Write "is correlated with" or "predicts".
- Report the CI. If ρ is modest (0.3–0.5), say "moderately correlated". That is still publishable, because the software-testing literature (Inozemtseva & Holmes 2014) shows coverage is often *only* weakly correlated with effectiveness.

### 3.2 Contribution list (exactly 3)

1. **AQS, a Trojan-independent, per-assertion security quality score.** It unifies coverage over expert-defined security signals (SRTLC), formal behavioral evasion resistance (AER, FPV) and functional-path coverage (SAPC, SPV). SAPC refines the SPV-based Security Coverage of [Eslami 2022] from whole-netlist nodes to the input-to-security-signal paths that matter for the threat model. *(Hedge: "to our knowledge, the first to combine FPV-based evasion and SPV-based path evidence into a single per-assertion security score". Keep "to our knowledge".)*
2. **Predictive validation on a real RISC-V core.** We build a 39-Trojan benchmark for Ibex covering 9 security modules and 6 TrustHub categories, written independently of the assertions. Using it, we test empirically whether formal quality computed before any Trojan exists predicts detection. We report rank correlation with confidence intervals and compare against coverage-only and mutation-style baselines. *(Hedge: claim "predictive validity on this benchmark", not "in general". This is the most defensible "new" element, because no prior assertion-quality metric in the hardware-security literature that I found reports such a correlation study.)*
3. **AQS-guided refinement closing the loop.** Gap reports (AER bypass traces, unobserved SPV paths) drive template-based generation of new assertions. On *held-out* Trojans and against a random-refinement control, this improves TDER/WTDR. *(Hedge: credit TDER/TPI to [ISCAS 2025]; claim the improvement only against the control.)*

### 3.3 Claims that must NOT appear

| Claim | Why it fails |
|---|---|
| "First SPV-based assertion metric" | Eslami 2022 / SCARF 2024 |
| "First per-assertion security score" | Eslami 2022 (SC per assertion, ranked) |
| "First Trojan evaluation of assertions on Ibex/RISC-V" | own ISCAS 2025; Iman NorCAS 2024; Chuah 2026 |
| "AER extends Chuah's free-XOR HT model" | Not verified. Cite Chockler 2001 and formal-FI work for the mechanism instead, until the Chuah PDF has been read. |
| "Proves AQS predicts detection" | Correlation is not proof. Write "empirically supports". |
| "Benchmark based on TrustHub Trojans" | Krieg 2023 critique. Write "categories follow the TrustHub taxonomy; all 39 Trojans are new RTL for Ibex". |
| "AER=1 means no Trojan can evade" | It holds only within the perturbation model (one XOR point per signal) and the oracle property. |

---

## 4. "Why this is not just X" defenses (one sentence each, plus required design fix)

**SRTLC vs Jasper COI coverage / AssertionForge COI metrics.**
> "Standard COI coverage divides by all RTL in the module, which rewards large functional checks. SRTLC's denominator is only the RTL that drives signals named in expert security properties (NS31A), including signals that were *not* translatable, so it measures how much of the security-relevant logic is observed, not how much logic is observed."
- *Fix to survive review:* compute SRTLC on **proof-core** as well as COI, and report both. Synopsys- and Cadence-sourced material notes that COI over-approximates what is actually verified. Proof-core makes SRTLC much harder to dismiss as "COI with a filter".

**AER vs mutation / formal fault injection (Chockler 2001; Jasper COV mutation; Zuccalà 2024).**
> "Mutation coverage asks whether *some* fixed, enumerated mutant falsifies an assertion. AER instead universally quantifies over *all* perturbation sequences of a security signal in a single FPV query, and judges the outcome against an independent *security* property rather than functional equivalence. So it measures whether an attacker who controls that signal can break the security goal while the assertion stays silent, and it returns the bypass trace when that is possible."
- *Fix to survive review:*
  - The "intended security property" must be a **separate oracle**: the original NS31A property, or a miter against an unperturbed copy restricted to security outputs such as privilege level, PMP grant or mepc. It must not be the assertion itself, otherwise AER is trivially 1.
  - A fully free XOR will almost always find *some* violation the assertion does not check. AER will then saturate at 0, become a constant, and be useless for ranking. Make AER **graded**: the fraction of security signals in the module (or in the assertion's own COI) whose free perturbation the assertion catches. Alternatively bound it per cycle (single-cycle or k-cycle glitch).
  - Report the AER distribution. If more than 80% of assertions get the same value, a reviewer will say the term is dead weight.

**SAPC vs Eslami 2022 Security Coverage (SPV).**
> "Security Coverage counts *all* netlist nodes that can reach an assertion output, so focused security checks score a few percent and the score depends on design size. SAPC fixes the target set to the expert-defined security signals and counts which primary-input-to-security-signal paths, proven functional by SPV, are observed by some assertion. That makes it a threat-model-scoped and size-independent refinement of the same idea from our group."
- *Critical fix:* if "path inside an assertion COI" is computed syntactically, SAPC collapses to COI coverage. Every assertion that reads signal s has the *entire* structural fan-in of s in its COI, so SAPC is trivially 100% for any referenced signal and SPV adds nothing. Define "monitored" as **SPV proves a functional (taint) path from input i through security signal s to the assertion's failure output**, which is exactly Eslami's SPV pair-check restricted to (i, s, assertion) triples. That is what makes SAPC non-trivial and keeps the "structural vs behavioral" story honest.

**AQS vs Assert-O ranking / CWE-CVSS scoring / Eslami ranking.**
> "Assert-O ranks properties by the *severity* of the vulnerability they relate to, and Eslami ranks by node reach alone. AQS ranks by *formal evidence of detection ability*, combining three independent formal checks. Severity is deliberately kept out of the score, as a descriptive CWE/CVSS table, so that the score cannot be inflated by labeling."
- *Fix:* justify equal weights by reporting rank stability under random weights (E4). Otherwise a reviewer will say "arbitrary weights".

---

## 5. Experiments that most strengthen the paper (6-page budget)

Ordered by value per column-inch. **E1–E5 are the top 5.**

**E1. Correlation done properly (mandatory, roughly 1/4 column + 1 small plot).**
- Use Spearman ρ and Kendall τ with **bootstrap 95% CIs** (10k resamples). Do not rely on Pearson only: AER is binary-ish and detection counts are small integers with ties.
- **Control for module.** An assertion can only detect Trojans in its own module's files, and modules have different Trojan counts (4–6). Use either (a) within-module normalized detection (detected / Trojans in module) or (b) partial correlation / mixed model with module as a random effect. Without this, the correlation may just reflect "modules with more Trojans have more detections".
- **Control for assertion "size"** (COI size or number of signals), following Inozemtseva & Holmes 2014. Show AQS still correlates after partialling out size. This is the single most likely reviewer attack.
- Add a **top-k precision** figure: of the top-quartile AQS assertions, what fraction detect at least one Trojan, compared with the bottom quartile. It is easy to read in one figure.

**E2. Baselines: does AQS beat cheaper metrics? (one small table)**
Report the predictive ρ for:
- (a) plain Jasper COI coverage;
- (b) proof-core coverage;
- (c) Eslami-style SC (whole-netlist nodes), the in-group baseline;
- (d) Jasper COV mutation coverage or a small set of hand-written mutants (mutation kill rate);
- (e) random ranking, shown as a permutation-test null distribution;
- (f) number of signals in the assertion.

If AQS ≥ all of them, contribution 1 is earned. If mutation beats AQS, report it honestly and position AQS as "comparable predictive power plus actionable gap reports (bypass traces, unobserved paths) that mutation does not give".

**E3. Held-out Trojans and a random-refinement control for the refinement claim (mandatory for contribution 3).**
- **Split** the 39 Trojans into a tuning set and a held-out set, stratified by category. Alternatively generate 10–15 *new* Trojans after ARS is frozen. ARS sees only AQRS gaps, not Trojans, but a reviewer will still suspect the templates were tuned to the same 39 files.
- **Control:** add the *same number* of template assertions on *randomly chosen* security signals. The claim is "AQS-guided > random-guided", not just "more assertions > fewer assertions".

**E4. Ablation and weight sensitivity (one table or one line plot).**
- Leave-one-component-out: ρ for AQS without SRTLC, without AER and without SAPC. This shows each term carries signal, and is the strongest answer to "why three metrics?"
- Weight robustness: sample weights from Dirichlet(1,1,1) (1000 draws) and report the median and IQR of Kendall τ between the resulting ranking and the equal-weight ranking, plus the ρ range.

**E5. AER vs mutation kill rate, and SAPC vs SC, on the same 68 assertions (compact two-panel scatter).**
- Show they are *related but not redundant*: moderate correlation, plus concrete disagreement cases. One example of an assertion that kills all mutants but has an AER bypass, and one that has high SC but low SAPC. One worked bypass trace in the text is worth more than a page of definitions.
- This directly defends the "why not just X" rows of §4.

Secondary experiments (only if space allows, otherwise put them in a technical report or appendix):
- **E6 Leave-one-module-out:** recompute ρ with each of the 9 modules removed. This shows no single module (e.g. controller with 4 logical modules sharing one file) drives the result. Report the min/max ρ in one sentence.
- **E7 Shared-file dependence:** DO, ETI, CF and MT share ibex_controller.sv and its 6 Trojans. Detection counts for these modules are therefore not independent observations. Report ρ with the controller modules collapsed into one, or use a cluster bootstrap by RTL file.
- **E8 Runtime/cost table:** FPV/SPV runtime per metric, as an ETS audience values practicality. Expect per-run cost to be the reviewer question for SPV.
- **E9 Category view:** detection by TrustHub category against the assertion→category descriptive table. Is the descriptive mapping consistent with actual detections?

**Statistical-power note.** With n = 68 the 95% CI half-width for ρ ≈ 0.4 is roughly ±0.2. Within-module analyses have n ≈ 5–10 per module, which is too few to report per-module ρ. Report pooled stratified estimates only.

---

## 6. Threat-model and benchmark wording that survives review

- "Our 39 Trojans follow the TrustHub category taxonomy (trigger/payload classes). They are new RTL modifications of Ibex, not TrustHub netlists. This avoids the validity issues reported for TrustHub designs [Krieg ICCAD 2023]."
- Independence: state *when* and *by whom* the Trojans were written relative to the AQS computation, e.g. "frozen and hashed before AQRS was run". Ideally a different author wrote them, or they come from the ISCAS 2025 benchmark generation process. Add a commit hash or date in the artifact.
- Detection semantics: CEX on trojaned RTL counts as detection only if the same assertion is Proven on clean RTL. That is already true (AVS), so say it explicitly. Also report Trojans whose CEX needs bounded depth > N, or which are undetermined, as a separate category rather than silently "not detected".

---

## 7. Suggested related-work paragraph skeleton (≈ 12 lines)

1. **Assertions as Trojan checkers.** Eslami et al. [2201.01130] and SCARF [2402.12162] introduced SPV-based Security Coverage per assertion. We refine its target set and add behavioral evidence and predictive validation.
2. **Formal coverage and mutation.** Chockler et al. [TACAS 2001]; Jasper COI, proof-core and mutation coverage; mutation-guided assertion refinement [2606.21451]. AER specializes this to a security oracle and a universally quantified perturbation.
3. **Security coverage via information flow.** Meza & Kastner [2304.08263]; Coşkun et al. ASP-DAC 2024. These are simulation- or VP-based and design-level, not per-assertion.
4. **Assertion completeness and HTs.** Fern & Cheng [DATE 2019].
5. **Formal HT detection on RISC-V.** Chuah et al. [MEMOCODE 2023; ISQED 2026]; HT-PGFV [Electronics 2024]; Iman et al. [NorCAS 2024]; own ISCAS 2025 and AutoTrans BEC 2026, which are the source of the assertions, TDER and TPI.
6. **Assets and severity.** LAsset [2601.02624], which identifies assets upstream of our work; NIST CSWP 45 for the descriptive CWE table.
7. **Benchmarks.** TrustHub with Krieg's critique [ICCAD 2023]; TrojanForge [2405.15184]; SENTAUR [2407.12352].
8. **Methodological precedent for "does coverage predict effectiveness".** Inozemtseva & Holmes [ICSE 2014]; Just et al. [FSE 2014]. These are software-testing papers, but citing one sentence of them signals statistical maturity to reviewers.

---

## 8. Items I could not verify (do not cite without checking)

- The Chuah MEMOCODE 2023 / ISQED 2026 free-XOR HT model. Only the abstract and the DBLP entry were accessible.
- The exact title and authors of the JKU DATE 2012 "guiding coverage metric" paper. Only a snippet was seen.
- The content of the MBMV 2025 "Coverage Metrics for Security Property Verification" paper. Only the title was seen.
- The content of Fern & Cheng ICCAD 2015. The citation was confirmed only via a CV listing.
- The exact venue printed on the Zuccalà et al. paper. It sits in a MEMOCODE folder of the ESWEEK 2024 proceedings, but the page does not state the venue.
- The SC detection results in SCARF beyond its abstract.
- The ETS 2027 CFP page limit and deadline. Not searched in this session; CLAUDE.md already flags them as unconfirmed.
