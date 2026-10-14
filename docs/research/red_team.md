# Red-Team Novelty Review: AQS Assertion Quality Ranker (target: IEEE ETS 2027)

Date: 2026-10-14. Written from the point of view of a hostile, well-read ETS reviewer.
Every citation below came from a web search or fetch run during this session. Where I could
only read an abstract or snippet, I say so. "Could not verify" means exactly that.

---

## 0. Bottom line

| Claim | Verdict |
|---|---|
| SRTLC (COI-based security line coverage) | **Partially overlapping, weak.** COI coverage is a standard formal-coverage metric that is known to over-count. The only new part is the NS31A-based denominator. |
| AER (XOR free-variable evasion check) | **Pre-empted in mechanism and in purpose.** Chuah/Appold/Leinmüller already use `sig_mod = sig XOR TROJ_input` with Jasper, specifically to *prove a property set detects every generic HT*. The only new part is the binary per-assertion label. |
| SAPC (SPV path coverage) | **Pre-empted by the co-author's own work** (Eslami, Ghasempouri, Pagliarini, ISQED 2022, and SCARF in IEEE TC 2024). It may also be **logically redundant with SRTLC** (see Attack 1). |
| AQS composite ranking | **Partially overlapping.** Multi-metric assertion ranking with rank aggregation already exists (Hertz, Pal, Offenberger, Vasudevan, ASP-DAC 2019, and the Vasudevan patent). An equal-weight mean of three correlated terms is not a contribution in itself. |
| "Formal score predicts Trojan detection" | **Appears novel as an explicit claim, but it is the most fragile part of the paper.** Prior work uses mutation or kill-ratio as the ground truth. Here the Trojans come from the same group and one of the metrics is the payload model itself. |

The paper's real novelty is narrow: **a security-signal-grounded, per-assertion composite,
checked empirically against a Trojan set on Ibex.** It survives only if (a) it frames Chuah and
Eslami as the direct ancestors, (b) it shows the three metrics are not redundant, and (c) the
validation statistics are honest.

---

## 1. Per-claim analysis

### 1.1 SRTLC: Security RTL Coverage (COI)

**Verdict: Partially overlapping.**

Closest prior art:
- **Jasper COV App (Cadence).** Commercial "Checker Coverage" already includes cone-of-influence, mutation and Proof Core analysis "to assess how completely the assertion set verifies the DUT". https://newstaging.cadence.com/zh_CN/home/tools/system-design-and-verification/formal-and-static-verification/jasper-gold-verification-platform/jaspergold-design-coverage-verification-app.html
- **Proof-core coverage patents (US 8826201 / 9177089).** These say outright that COI-based coverage "may overestimate the amount of coverage provided by a property". The proof core (an UNSAT-core subset of the COI) is the accepted fix. https://patents.google.com/patent/US8826201 , https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/9177089
- **DVCon, "Automate and Accelerate RISC-V Verification by Compositional Formal Methods".** It already reports COI and proof-core coverage per RISC-V core under JasperGold. https://dvcon-proceedings.org/wp-content/uploads/Automate-and-Accelerate-RISC-V-Verification-by-Compositional-Formal-Methods.pdf
- **AMD/DVCon "second generation completeness analysis of formal assertions on RISC-V cores".** It flags that even proof-core scoring marks logic as covered once any single use of it is checked. https://dvcon-proceedings.org/wp-content/uploads/second-generation-completeness-analysis-of-formal-assertions-on-compatibility-of-risc-v-cores-1.pdf
- **Athavale, Ma, Hertz, Vasudevan, DAC 2014: code coverage of assertions** using static and dynamic analysis of the RTL source (seen via the IEEE CEDA listing only). https://ieee-ceda.org/media/1341
- **Information-flow coverage metrics.** Meza & Kastner, arXiv 2304.08263 (https://arxiv.org/abs/2304.08263) and Coskun, Ahmadi-Pour, Hassan, Drechsler, ASP-DAC 2024 / MBMV 2024, "Security Coverage Metrics for Information Flow at the System Level" (https://www.aspdac.com/aspdac2024/archive/pdf/9C-3.pdf ; https://www.dfki.de/web/forschung/projekte-publikationen/publikation/14699). These are security-specific structural coverage metrics.

**Most damaging reviewer sentence:**
> "SRTLC is COI coverage restricted to a hand-picked subset of lines. COI coverage is documented by the vendor's own patents to overestimate coverage, and the authors use JasperGold, which ships proof-core coverage, yet they neither use it nor justify why not."

### 1.2 AER: Assertion Evasion Resistance (XOR free variable + FPV)

**Verdict: Pre-empted in mechanism and in purpose. The per-assertion binary score is a thin delta.**

Closest prior art:
- **Chuah, Appold, Leinmüller, "Reliable Hardware Trojan Formal Verification", RISC-V Summit Europe 2025, Paris.** I fetched and read the full abstract text. Quoting the content: they "developed a generic HT model able to model an arbitrary HT modifying signal values. The HT model uses a new free input (TROJ_input)... `assign sig1_mod = sig1 XOR TROJ_input`... dedicated properties ensure every possible triggered HT... is detected through our property set". They also claim to be "the first to suggest using formal verification also to verify the completeness of a property set for reliable HT detection". Tool: Cadence Jasper. Target: a 32-bit 4-stage RISC-V core. **This is AER's mechanism and AER's purpose.** https://riscv-europe.org/summit/2025/media/proceedings/2025-05-14-RISC-V-Summit-Europe-P1.1.07-CHUAH-abstract.pdf
- **Chuah, Appold, Leinmüller, "Reliable Hardware Trojan Detection for RISC-V Processors using Formal Verification and Automation", ISQED 2026.** I verified it exists (ISQED 2026 archive page). Abstract: a "complete property set for detecting each inserted HT modifying the Program Counter at an arbitrary position"; connection properties plus design-intent properties. The abstract does not mention XOR, free variables, JasperGold or Ibex; those details come from the Summit 2025 abstract above. https://isqed.org/English/Archives/2026/Technical_Sessions/79.html
- **Kupferman, Li, Seshia, "A Theory of Mutations with Applications to Vacuity, Coverage, and Fault Tolerance", FMCAD 2008.** This gives the formal basis: a mutation of the circuit that a property does not detect means a coverage gap. Inserting a free XOR is a universally quantified mutation. https://people.eecs.berkeley.edu/~sseshia/pubs/b2hd-kupferman-fmcad08.html
- **Das, Banerjee, Dasgupta, "A generalized theory for formal assertion coverage", ATS 2012.** It unifies coverage based on fault injection into the implementation with coverage based on mutating the specification. https://repository.ias.ac.in/101627
- **Fern & Cheng, "Evaluating Assertion Set Completeness to Expose Hardware Trojans and Verification Blindspots", DATE 2019.** It uses behaviour the assertion set leaves uncovered to expose Trojans and verification blind spots. https://past.date-conference.com/proceedings-archive/2019/html/0322.html . Related: Fern & Cheng, ICCAD 2015, Trojans in unspecified functionality via mutation testing (the authors' "IEEE 7372619").

**Most damaging reviewer sentence:**
> "AER is the generic XOR-free-input Trojan model of Chuah et al. (RISC-V Summit Europe 2025; ISQED 2026), whose stated goal is already to formally verify the completeness of a property set for HT detection. Reporting a 0/1 per assertion rather than per property set is bookkeeping, not a new technique."

Also note: an XOR free input on a signal is the standard cut-point / `stopat` abstraction, which JasperGold supports natively. A reviewer will say the paper re-implements `stopat` by hand.

### 1.3 SAPC: Security Assertion Path Coverage (SPV)

**Verdict: Pre-empted, by a co-author's own papers.**

Closest prior art:
- **Eslami, Ghasempouri, Pagliarini, "Reusing Verification Assertions as Security Checkers for Hardware Trojan Detection", ISQED 2022 (arXiv 2201.01130).** I read the full text. It defines Security Coverage = reachable nodes / total nodes, **per assertion**, using JasperGold SPV with (origin = every netlist node, destination = assertion output). It then adds an **assertion selection/ranking flow** (fixed- and dynamic-threshold strategies) over 108 OpenTitan assertions. https://arxiv.org/abs/2201.01130
- **Eslami, Ghasempouri, Pagliarini, "SCARF", IEEE Trans. Computers 2024 (DOI 10.1109/TC.2024.3449082).** This extends Security Coverage. Per the search snippet it is unioned across assertions and can also be applied per assertion. https://arxiv.org/abs/2402.12162
- **Rajendran et al. / "Hardware Trojan Detection through Information Flow Security Verification" (arXiv 1803.04102).** SPV/taint-based Trojan detection, including where SPV fails (AES-T2000/T2100 power side-channel leaks). https://arxiv.org/pdf/1803.04102
- **HT-PGFV (Qin et al., Electronics 13(21):4286, 2024).** It already uses "influence-cone-analysis-based Trojan path identification" plus information-flow mining to generate and check Trojan properties. https://www.mdpi.com/2079-9292/13/21/4286/htm

**Two technical problems a reviewer with SPV experience will raise:**
1. **The paper calls SPV "structural"; Eslami et al. call it functional.** Eslami et al. (footnote 1) say SPV proves *functional* paths, "in contrast with structural paths", meaning paths that can be exercised by valid inputs. The project framing ("AER = behavioural, SAPC = structural") therefore misdescribes the tool. If SPV paths are functional, the claimed orthogonality between AER and SAPC is weaker than stated.
2. **SAPC may be close to tautological (see Attack 1).** Any path from an input to a security signal S lies in the COI of every assertion that reads S, or reads anything S drives.

**Most damaging reviewer sentence:**
> "SAPC is the SPV-based Security Coverage metric of Eslami, Ghasempouri and Pagliarini (ISQED 2022; IEEE TC 2024), with the destination changed from the assertion output to a security signal. Since one of those authors co-authors this submission, the claim of novelty is not credible."

### 1.4 AQS composite ranker

**Verdict: Partially overlapping.**

Closest prior art:
- **Hertz, Pal, Offenberger, Vasudevan, "A Figure of Merit for Assertions in Verification", ASP-DAC 2019.** It notes "there is no metric to evaluate assertions", ranks RTL assertions by importance and complexity, and validates the top-ranked assertions downstream. https://experts.illinois.edu/en/publications/a-figure-of-merit-for-assertions-in-verification/
- **US 9075935, "Merit-based characterization of assertions in hardware design verification" (Vasudevan group).** It covers importance, complexity, coverage and expectedness, aggregated via rank aggregation with Kendall-tau evaluation. It notes the metrics correlate weakly, which is its stated reason for aggregating them. https://patents.google.com/patent/US9075935
- **Eslami et al., ISQED 2022.** An assertion selection flow that trades security coverage against overhead (above).
- **Witharana et al.** A clustering-based assertion selection that maximises fault coverage and reportedly beats GoldMine and HARM ranking (seen only in an NSF PAR listing; I did not verify the exact paper title). https://par.nsf.gov/search/author:%22Witharana,%20Hasini%22
- **HARM / GoldMine.** User-defined ranking metrics for mined assertions. https://arxiv.org/pdf/2601.02248 (survey)

**Most damaging reviewer sentence:**
> "Ranking assertions with an aggregate of several merit scores was done by Hertz et al. (ASP-DAC 2019) with principled rank aggregation. Here the weights are an unjustified 1/3 each, there is no ablation, and the paper does not check whether the three terms are even independent."

### 1.5 Validation claim: AQS predicts Trojan detection

**Verdict: Appears novel as an explicit correlation claim on a security-assertion set. Methodologically it is the most vulnerable part of the paper.**

Related prior art:
- **Mutation kill-ratio as the quality ground truth.** AssertLLM2 (arXiv 2605.27472) states "FPV outcomes alone do not fully characterize assertion quality" and adds a mutation kill ratio. Arcane (arXiv 2605.10107) argues mutation detection reflects assertion quality more directly than structural metrics. https://arxiv.org/pdf/2605.27472 , https://arxiv.org/pdf/2605.10107
- **Software-testing literature.** Coverage correlates only moderately with fault-detection effectiveness (ICST 2019: statement coverage correlates with mutation score more strongly than assertion coverage does). A reviewer will expect this result to transfer. https://computer.org/csdl/proceedings-article/icst/2019/173600a252/1aDT5pSd2GQ
- **The authors' own prior work already reports Trojan detection for translated assertions on Ibex using TPI and a "Trojan Detection Efficiency Ratio"**, with LLM (ChatGPT-4) generated Trojans and 100% detection (Imtiaz, Reinsalu, Ghasempouri, arXiv 2502.10194; the TPI/TDER detail comes from search-result summaries). https://arxiv.org/abs/2502.10194 . Iman, Ahmadi-Pour, Drechsler, Ghasempouri, NorCAS 2024, also reports "all inserted Trojans" caught. https://ws.lib.ttu.ee/digibase/en/Publ/Item/ffcc6486-8fca-4db5-928b-7ce621bf21dc . **TDER, WTDR and TPI are therefore not new, and 100% detection in prior work makes it hard to show any variance for a correlation now.**

**Most damaging reviewer sentence:**
> "Correlation is reported over n=68 assertions whose detection counts against 39 author-written Trojans are mostly 0 or 1. AER is literally an exhaustive generic-Trojan check, so its correlation with Trojan detection is guaranteed by construction, not discovered."

---

## 2. Top 8 methodological attacks

| # | Attack | Severity | What would neutralize it |
|---|---|---|---|
| 1 | **SAPC is redundant with SRTLC, or tautological.** A COI is the transitive fan-in. Every path from a primary input to a security signal S is inside COI(A) whenever A references S or anything downstream of S. So SAPC = 100% for every referenced signal and 0% otherwise, which reproduces SRTLC's signal-level information. | **Fatal** if true | Give a formal definition where SAPC ≠ f(SRTLC). Show a concrete Ibex case of a path to S outside every COI while S is still in SRTLC's numerator. Report the SRTLC–SAPC Pearson/Spearman correlation across all 68 assertions (it must be well below 1). Better: switch to proof-core-restricted paths or taint-to-assertion-output as in Eslami. |
| 2 | **AER circularity: what is the "intended security property P"?** If P is the assertion itself, AER is always 1 (an assertion cannot violate itself silently). If P is a separate golden spec, then where do those 68+ golden properties come from, and why are they not the assertions? | **Fatal** if unclear | State P precisely: for example a non-interference or "signal unchanged unless authorised" property written independently of A. Show the oracle miter. Report how many AER=0 cases exist and give one trace in the paper. |
| 3 | **AER = Chuah's generic HT model plus standard `stopat`/cut-point.** It is not a new technique. Binary 0/1 also discards information, so most assertions will tie. | **Major** | Cite Chuah RISC-V Summit 2025 and ISQED 2026 as the direct origin. Make AER graded: the fraction of XOR injection points per assertion COI that are caught, or the bounded depth to the CEX. Show the AER distribution is not degenerate. |
| 4 | **The Trojan benchmark is not independent.** The same authors wrote the assertions (via translation) and the 39 Trojans. The prior paper used LLM-generated Trojans and caught 100%. Krieg (ICCAD 2023) found only 3 of 83 TrustHub benchmarks are real Trojans. https://repositum.tuwien.at/handle/20.500.12708/191214 | **Major** | Freeze the Trojans before computing AQS (timestamped commit). Add third-party Trojans, e.g. the Hayashi & Ruggiero ChatGPT/TrustHub-taxonomy dataset (Data 2024, https://doaj.org/article/350c2b1dc37a43e0831328c012d3346a) or Hack@DAC-style bugs. Report stealth (TPI) per Trojan, and show some Trojans are missed. |
| 5 | **The statistics are weak.** n=68 assertions across 9 modules. Detection counts are small integers with many ties and zeros. Pearson assumes linearity and normality. Module confounding is likely: high-CVSS modules get both more Trojans and more assertions. | **Major** | Use Spearman/Kendall with bootstrap CIs and exact p-values. Use a per-module mixed model or a within-module rank correlation. Report a leave-one-module-out analysis and a baseline predictor (for example, just counting the signals an assertion references, or COI size). If AQS does not beat COI size, the ranker is pointless. |
| 6 | **COI coverage is a known weak proxy.** Cadence's own patents say it overestimates, and proof core is the standard fix. SRTLC also inherits the NS31A denominator: signals NS31A experts did not think about are invisible by construction. | **Major** | Recompute SRTLC with proof-core coverage (Jasper COV supports this) and show that the ranking is stable. Discuss the NS31A-denominator blind spot explicitly as a limitation. |
| 7 | **Refinement is evaluated on the same 39 Trojans that exposed the gaps.** This is train-on-test, so the improvement in TDER/WTDR/AAD is guaranteed rather than measured. The templates (`signal != 0 \|-> sec == $past(sec)`) also look like over-constraining checks that may be vacuous or trivially triggered. | **Major** | Hold out Trojans: split 39 into disjoint refinement and evaluation sets, or generate a fresh set after refinement. Report how many refined assertions pass the FPV gate, and how many are vacuous. Show refinement is driven by the AQS gaps, not the TES gaps. |
| 8 | **Unequal weights, no ablation, and the CVSS/CWE mapping is ad hoc.** The CWE-to-module table maps "Debug Feature Enabled in Production" to instruction execution, and "Insecure Defaults" to register update. CVSS ranges are attached to CWEs, which CVSS does not score (CVSS scores vulnerabilities, not weaknesses). The PMP Availability, MA Privilege and RU Privilege categories are missing, which biases category tables. | **Minor** (CWE is descriptive only now) / **Major** if weights are kept | Ablation: the correlation of each metric alone vs AQS. Justify equal weights, or learn them with cross-validation. Drop CVSS numbers, or use NIST CSWP 45 threat/sensitivity metrics, which are defined for CWE/CAPEC. |

Additional minor attacks:
- **No runtime or scalability numbers.** ETS reviewers care about test cost, and 144 FPV/SPV runs need wall-clock figures.
- **One core (Ibex), with 4 of 9 "modules" sharing one file (`ibex_controller.sv`).** That shrinks the effective number of independent modules to 6.
- **Prior results to rebut.** The authors' 2025 arXiv preprint reported 100% detection. If assertions were already catching everything, why does the set need ranking? The paper must explain this.
- **Terminology.** "TDER = Trojan Detection Efficiency *Rate*" here vs "*Ratio*" in arXiv 2502.10194. Do not present TPI/TDER as new.

---

## 3. Does "Chuah et al., ISQED 2026" exist?

**Yes, it exists.** "Reliable Hardware Trojan Detection for RISC-V Processors using Formal Verification and Automation", Czea Sie Chuah (TU Munich), Christian Appold, Tim Leinmüller (DENSO AUTOMOTIVE Deutschland), ISQED 2026 technical session page: https://isqed.org/English/Archives/2026/Technical_Sessions/79.html . A proceedings ToC lists it at p. 163: https://www.proceedings.com/content/086/086256webtoc.pdf . I found no IEEE Xplore DOI.

- **The ISQED abstract itself does not mention XOR, free variables, JasperGold or Ibex.** Its focus is a complete property set for PC-modifying Trojans plus auto-generated signal-connection properties, with proofs under 24 hours.
- **The XOR free-input generic HT model is confirmed in the same group's RISC-V Summit Europe 2025 abstract** (full text read, quoted in §1.2), using Cadence Jasper on a 32-bit 4-stage RISC-V core (not Ibex as far as stated). Cite **both**.
- **Correction to the brief.** The "Formal Verification of Security Properties on RISC-V Processors" paper by Chuah, Appold, Leinmueller that I found is in **MBMV 2024** (27th workshop, Kaiserslautern, 14–15 Feb 2024; https://www.vde-verlag.de/proceedings-en/456267021.html), **not MEMOCODE 2023**. I could not verify any MEMOCODE 2023 version. Fix this in the bibliography.
- The Summit abstract cites Iman et al. NorCAS 2024 (Tara Ghasempouri is a co-author) as prior work that "doesn't target reliably detecting each inserted HT". So the Chuah group already knows this group's work, and a reviewer from that group is plausible.

---

## 4. How to reposition, so the paper survives

1. **Concede the ancestry openly.** "AER instantiates the generic XOR HT model of Chuah et al. [Summit'25, ISQED'26] at per-assertion granularity. SAPC adapts the SPV security-coverage of Eslami et al. [ISQED'22, TC'24] from Trojan-payload nodes to expert-defined security signals."
2. **Make the contribution the *combination plus validation*.** That means the claim that per-assertion behavioural and path scores, grounded in expert security signals, predict detection better than naive baselines (COI size, number of referenced signals, proof-core coverage). Without a baseline comparison the central claim is unfalsifiable.
3. **Prove the metrics are not redundant.** Give the pairwise correlation matrix and an ablation.
4. **Fix the validation design.** Freeze the Trojans first, add external Trojans, use rank statistics with CIs, and hold out Trojans for the refinement stage.
5. **Correct the SPV "structural" wording,** or justify it against Eslami's footnote that SPV finds functional paths.

---

## Sources consulted (all retrieved this session)

- Chuah et al., ISQED 2026: https://isqed.org/English/Archives/2026/Technical_Sessions/79.html
- Chuah et al., RISC-V Summit Europe 2025 abstract: https://riscv-europe.org/summit/2025/media/proceedings/2025-05-14-RISC-V-Summit-Europe-P1.1.07-CHUAH-abstract.pdf
- Chuah et al., MBMV 2024: https://www.vde-verlag.de/proceedings-en/456267021.html
- Proceedings ToC listing Chuah: https://www.proceedings.com/content/086/086256webtoc.pdf
- Eslami, Ghasempouri, Pagliarini, ISQED 2022: https://arxiv.org/abs/2201.01130 ; https://isqed.org/English/Archives/2022/Technical_Sessions/26.html
- SCARF (IEEE TC 2024): https://arxiv.org/abs/2402.12162
- Imtiaz, Reinsalu, Ghasempouri, arXiv 2502.10194: https://arxiv.org/abs/2502.10194
- Iman et al., NorCAS 2024: https://ws.lib.ttu.ee/digibase/en/Publ/Item/ffcc6486-8fca-4db5-928b-7ce621bf21dc
- Fern & Cheng, DATE 2019: https://past.date-conference.com/proceedings-archive/2019/html/0322.html ; https://vsdl.hkust.edu.hk/papers/NICOLE/NICOLE-DATE-2019.pdf
- Hertz, Pal, Offenberger, Vasudevan, ASP-DAC 2019: https://experts.illinois.edu/en/publications/a-figure-of-merit-for-assertions-in-verification/
- US 9075935 (merit-based assertion characterization): https://patents.google.com/patent/US9075935
- Athavale et al., DAC 2014 (via CEDA): https://ieee-ceda.org/media/1341
- COI/proof-core patents: https://patents.google.com/patent/US8826201 ; https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/9177089
- Jasper COV App: https://newstaging.cadence.com/zh_CN/home/tools/system-design-and-verification/formal-and-static-verification/jasper-gold-verification-platform/jaspergold-design-coverage-verification-app.html
- DVCon compositional RISC-V formal (COI/proof core numbers): https://dvcon-proceedings.org/wp-content/uploads/Automate-and-Accelerate-RISC-V-Verification-by-Compositional-Formal-Methods.pdf
- DVCon completeness analysis RISC-V: https://dvcon-proceedings.org/wp-content/uploads/second-generation-completeness-analysis-of-formal-assertions-on-compatibility-of-risc-v-cores-1.pdf
- Kupferman, Li, Seshia, FMCAD 2008: https://people.eecs.berkeley.edu/~sseshia/pubs/b2hd-kupferman-fmcad08.html
- Das, Banerjee, Dasgupta, ATS 2012: https://repository.ias.ac.in/101627
- Meza & Kastner, IF coverage: https://arxiv.org/abs/2304.08263
- Coskun et al., ASP-DAC 2024 / MBMV 2024: https://www.aspdac.com/aspdac2024/archive/pdf/9C-3.pdf ; https://www.dfki.de/web/forschung/projekte-publikationen/publikation/14699
- IF security verification / SPV taint: https://arxiv.org/pdf/1803.04102
- HT-PGFV: https://www.mdpi.com/2079-9292/13/21/4286/htm
- Krieg, "Reflections on Trusting TrustHUB", ICCAD 2023: https://repositum.tuwien.at/handle/20.500.12708/191214
- Hayashi & Ruggiero LLM Trojan dataset: https://doaj.org/article/350c2b1dc37a43e0831328c012d3346a
- AssertLLM2: https://arxiv.org/pdf/2605.27472 ; Arcane: https://arxiv.org/pdf/2605.10107
- ICST 2019 coverage vs mutation score: https://computer.org/csdl/proceedings-article/icst/2019/173600a252/1aDT5pSd2GQ
- Assertion mining survey (HARM/GoldMine): https://arxiv.org/pdf/2601.02248
- Witharana et al. (NSF PAR listing): https://par.nsf.gov/search/author:%22Witharana,%20Hasini%22
- Rogers et al., Security Properties for Open-Source Hardware Designs: https://arxiv.org/abs/2412.08769
- UPEC: https://arxiv.org/pdf/1812.04975
