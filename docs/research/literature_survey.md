# Literature Survey: Formal Security-Assertion Quality Metrics and Trojan Evaluation

Sweep date: 2026-10-14. Target: IEEE ETS 2027 (6 pages).

**Verification policy.** Every entry below came from a web search or fetch result in this session, and each URL was returned by that search. Where a field (venue, year, authors, exact title) was not visible in the result, it is marked **unverified**. Most summaries are based on abstracts or indexed snippets, not full texts. Re-check every BibTeX entry against IEEE Xplore, ACM DL, or DBLP before submission.

Legend for relevance: SRTLC = COI coverage of security-critical RTL; AER = free-XOR FPV evasion resistance; SAPC = SPV structural path coverage; AQS = composite ranker and its validation; BENCH = Trojan benchmark.

---

## A. Property/assertion coverage and quality in formal verification

### A1. Classic formal coverage, vacuity, and sanity checks

1. **Y. Hoskote, T. Kam, P.-H. Ho, X. Zhao, "Coverage Estimation for Symbolic Model Checking," DAC 1999, pp. 300-305.** DOI 10.1145/309847.309936 (from a Crossref reference list; abstract not retrieved).
   URL: https://api.crossref.org/works/10.1007%2F3-540-45614-7_3 (citing record) ; ASP-DAC'06 related-work slide: https://www.aspdac.com/aspdac2006/archives/pdf/1A-1.pdf
   What: The first state-coverage metric for model checking. It reports which design states the verified properties actually constrain.
   Relevance: Historical root of "how much of the design does a proven property set cover". It is the ancestor of SRTLC.

2. **H. Chockler, O. Kupferman, R. Kurshan, M. Y. Vardi, "A Practical Approach to Coverage in Model Checking," CAV 2001.**
   URL: https://www.cs.huji.ac.il/~ornak/publications/cav01.pdf
   What: Measures coverage by mutating the *system* and checking whether the specification still passes. Uncovered parts are those whose mutation goes undetected. Gives several coverage definitions for LTL/automata specifications.
   Relevance: **Conceptually closest classical precedent for AER.** AER injects a free XOR perturbation into a security signal and asks whether the assertion still passes, which is system-mutation coverage restricted to security signals. Must cite.

3. **H. Chockler, O. Kupferman, M. Y. Vardi, "Coverage Metrics for Temporal Logic Model Checking," TACAS 2001 (journal version later).**
   URL: https://www.cs.huji.ac.il/~ornak/publications/tacas01.pdf ; https://cris.huji.ac.il/en/publications/coverage-metrics-for-temporal-logic-model-checking-14/
   What: Separates state-based from logic-based coverage and system from environment, and gives algorithms to compute uncovered parts.
   Relevance: Formal grounding for SRTLC (state/signal coverage) and AER (mutation coverage).

4. **H. Chockler, J. Y. Halpern, O. Kupferman, "What Causes a System to Satisfy a Specification?" (arXiv cs/0312036; journal version unverified).**
   URL: https://arxiv.org/abs/cs/0312036
   What: Links coverage to Halpern-Pearl causality and defines a quantitative degree of *responsibility* of each state for a property being satisfied. Reports that the coverage metrics found escaped bugs in industrial practice.
   Relevance: A graded, per-element alternative to binary covered/uncovered. It could justify making AER or SRTLC_contrib non-binary.

5. **O. Kupferman, M. Y. Vardi, "Vacuity Detection in Temporal Model Checking," STTT (DOI 10.1007/s100090100062)**. The venue is inferred from the DOI prefix and is **unverified**. Extends I. Beer et al.'s vacuity and "interesting witness" work to CTL*.
   URL: https://unpaywall.org/10.1007%2FS100090100062 ; https://cris.huji.ac.il/en/publications/vacuity-detection-in-temporal-model-checking-13/
   What: A specification is vacuous if some subformula does not affect its truth. Interesting witnesses show that a property holds non-trivially.
   Relevance: Justifies the "Proven + non-vacuous" gate inherited from AVS. A vacuous assertion trivially gets AER=1, so AER needs a vacuity precondition.

6. **R. Armoni, L. Fix, A. Flaisher, O. Grumberg, N. Piterman, A. Tiemeyer, M. Y. Vardi, "Enhanced Vacuity Detection in Linear Temporal Logic," CAV 2003.**
   URL: https://orna.cswp.cs.technion.ac.il/wp-content/uploads/sites/72/2022/12/cav03-vacuity.pdf
   What: Industrial-strength LTL vacuity definitions and algorithms (Intel).
   Relevance: Same as above. Supports the non-vacuity precondition for AER.

7. **O. Kupferman, "Sanity Checks in Formal Verification," CONCUR 2006, LNCS 4137, pp. 37-51.**
   URL: https://cris.huji.ac.il/en/publications/sanity-checks-in-formal-verification/ ; https://www.cs.huji.ac.il/~ornak/publications/concur06babs.txt
   What: Surveys vacuity and coverage as the two sanity checks applied after a successful proof. Both are computed by re-verifying a mutated specification (vacuity) or mutated system (coverage).
   Relevance: **Best single citation framing AQS.** "A proof passed, but how good is the property?" is exactly the paper's question. AER is a security-targeted coverage check in this taxonomy.

8. **K. Claessen, "A Coverage Analysis for Safety Property Lists," FMCAD 2007, pp. 139-145.**
   URL: https://research.chalmers.se/en/publication/67786
   What: Design-independent analysis that finds "forgotten cases" where a property list leaves an output unconstrained at some time. It separates intended from unintended under-specification.
   Relevance: Analogue of the SAPC/TCFC "blind spot" idea at the specification level.

9. **D. Große, U. Kühne, R. Drechsler, "Estimating Functional Coverage in Bounded Model Checking," DATE 2007** (journal version: "Analyzing Functional Coverage in Bounded Model Checking," IEEE TCAD 27(7), 2008).
   URL: https://agra.informatik.uni-bremen.de/doc/konf/07date_coverage.pdf
   What: Generates a coverage property per important signal. A failing coverage property yields a trace of an uncovered scenario. Demonstrated on a RISC CPU, where the gaps were closed to 100%.
   Relevance: A signal-centric coverage metric with gap traces. It closely parallels AER's per-signal check and the ARS refinement loop.

10. **Cadence, "Formal verification coverage metrics for circuit design properties," US Patent 8,826,201 (also US 9,177,089). Patent, not peer-reviewed.**
    URL: https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/8826201
    What: Argues that COI-based coverage *over-estimates* what a property checks. It narrows the COI to a "proof core" and defines coverage as RTL statements in the proof core over all statements.
    Relevance: **Directly challenges SRTLC's COI choice.** Reviewers may ask why SRTLC uses COI rather than proof-core coverage, which JasperGold also reports. Address this explicitly or switch to proof core.

### A2. Assertion mining, ranking, and quality evaluation (pre-LLM)

11. **S. Vasudevan et al., "GoldMine: Automatic assertion generation using data mining and static analysis," DATE 2010** (IEEE Xplore 5457129; full author list **unverified** from the result).
    URL: https://ieeexplore.ieee.org/document/5457129 ; DVCon companion: https://dvcon-proceedings.org/wp-content/uploads/goldmine-automatic-assertion-generation-and-coverage-closure-in-design-validation.pdf
    What: Mines likely invariants from simulation, filters them with formal verification, then ranks the survivors with designer feedback.
    Relevance: Earliest "generate, prove, rank" pipeline. Prior art for the ranking idea behind AQS.

12. **S. Hertz (M.S. thesis, UIUC, 2013, advisor S. Vasudevan), plus US Patent 9,075,935 "Merit-based characterization of assertions in hardware design verification."**
    URL: https://www.ideals.illinois.edu/items/55261 ; https://patents.google.com/patent/US9075935
    What: Defines assertion importance, complexity, coverage, and expectedness, and aggregates them into a single rank.
    Relevance: **Prior art for a multi-term composite assertion score (functional, not security).** AQS should be positioned as the security-oriented, formally computed analogue.

13. **T. Ghasempouri, S. Payandeh Azad, B. Niazmand, J. Raik, "An automatic approach to evaluate assertions' quality based on data-mining metrics," ITC-Asia 2018, pp. 61-66.** DOI 10.1109/ITC-Asia.2018.00021.
    URL: https://ws.lib.ttu.ee/digibase/en/Publ/Register/Item/166817
    What: Ranks mined assertions with data-mining interestingness metrics as a cheaper proxy for mutation-analysis quality.
    Relevance: Co-author's own prior work on assertion-quality ranking (functional). Cite it to show lineage.

14. **S. Germiniani, G. Pravadelli, "HARM: A Hint-Based Assertion Miner," IEEE TCAD, 2022.**
    URL: https://iris.univr.it/handle/11562/1079506
    What: Mines LTL assertions from traces using user hints (templates, propositions, *ranking metrics*).
    Relevance: Shows that assertion-ranking metrics are standard in mining. Contrast with AQS, which uses formal rather than trace-statistical ranking.

15. **Property qualification by mutation analysis plus model checking (Univ. Verona, 2009)** (authors and venue **unverified**).
    URL: https://iris.univr.it/handle/11562/334962
    What: Evaluates property coverage and over-specification, and detects vacuous properties, using mutation analysis complemented by model checking.
    Relevance: Precedent for mutation-based property quality, which is AER's mechanism family.

16. **H. Witharana, Y. Lyu, S. Charles, P. Mishra, "A Survey on Assertion-based Hardware Verification," ACM Computing Surveys 54(11s), 2022.**
    URL: https://www.cise.ufl.edu/research/cad/Publications/csur22.pdf
    What: A survey of assertion definition, automatic generation, and activation for functional and non-functional (security) requirements.
    Relevance: The survey citation for ABV background in §1/§2.

17. **Automatic Assertion Mining in Assertion-Based Verification: Techniques, Challenges, and Future Directions (arXiv 2601.02248, 2026)** (authors and venue **unverified**).
    URL: https://arxiv.org/pdf/2601.02248
    What: A recent survey of assertion mining (GoldMine, HARM, and others).
    Relevance: Recent survey anchor for assertion-quality discussion.

18. **L. Inozemtseva, R. Holmes, "Coverage Is Not Strongly Correlated with Test Suite Effectiveness," ICSE 2014** (Distinguished Paper; later Most Influential Paper).
    URL: https://cs.ubc.ca/~rtholmes/papers/icse_2014_inozemtseva.pdf
    What: Across about 31k Java test suites, coverage shows only low-to-moderate correlation with mutant detection once suite size is controlled for.
    Relevance: **Methodological warning for the AQS-vs-detection Pearson claim.** Control for confounders (for example, assertion count per module, module size), and report Spearman as well as Pearson. Software domain, but reviewers know it.

### A3. LLM-generated assertion quality (2023-2026)

19. **R. Kande et al., "LLM-assisted Generation of Hardware Assertions," arXiv 2306.14027, 2023.**
    URL: https://arxiv.org/abs/2306.14027v1
    What: Evaluates LLM generation of *security* SVAs from natural-language prompts on a benchmark with reference assertions.
    Relevance: Security-SVA generation baseline. It judges correctness, not quality ranking.

20. **NVIDIA, "FVEval: Understanding Language Model Capabilities in Formal Verification of Digital Hardware," arXiv 2410.23299, 2024.**
    URL: https://arxiv.org/abs/2410.23299
    What: NL2SVA and Design2SVA benchmarks, with formal-equivalence-based scoring of generated SVAs.
    Relevance: Shows that current evaluation is correctness-centric. AQS adds quality beyond correctness.

21. **AssertionBench (UIC and Microsoft), arXiv 2406.18627, 2024** (author list **unverified**).
    URL: https://www.arxiv.org/pdf/2406.18627
    What: 100 OpenCores designs with GoldMine/HARM formally verified reference assertions for benchmarking LLM generation.
    Relevance: Correctness benchmark. Contrast point for AQS.

22. **W. Fang, Z. Yan, et al., "AssertLLM: Generating (and Evaluating) Hardware Verification Assertions from Design Specifications via Multi-LLMs," ASP-DAC 2025, arXiv 2402.00386.**
    URL: https://arxiv.org/abs/2402.00386v1 ; https://www.aspdac.com/aspdac2025/archive/pdf/4D-1.pdf
    What: Spec-to-SVA generation with an evaluation flow.
    Relevance: Generation-side related work (contrast only).

23. **AutoAssert / TrustAssert (Zhai et al. per CLAUDE.md; author list unverified in search), DATE 2026.**
    URL: https://past.date-conference.com/proceedings-archive/2026/DATA/159.pdf
    What: Builds a 110K formally verified assertion dataset via equivalence checking and fine-tunes LLMs on it. Reports "non-trivial assertion ratio" as a quality signal.
    Relevance: Name-collision risk with "AutoAssert-RV". Generation, not ranking. Cite briefly.

24. **HierSVA: "A Data Synthesis Pipeline, Dataset, and Benchmark for LLM-Driven Hierarchical Hardware Formal Verification," arXiv 2606.13706, June 2026** (authors **unverified**, Univ. of Washington).
    URL: https://www.alphaxiv.org/abs/2606.13706
    What: Scores assertions on six axes: syntax, proof rate, vacuity, spec faithfulness, **mutation coverage**, and **formal-core coverage**. LLM assertion sets cover only 36.2% of the formal core and catch 70.2% of injected faults.
    Relevance: **Closest 2026 work on multi-axis formal assertion quality.** It is functional, not security, and has no SPV or Trojan validation. It overlaps SRTLC (formal-core coverage) and AER (mutation kill). Must cite and differentiate.

25. **R. Krishnamurthy, D. Chitnis, T. Prodromakis, "Closing the Loop on LLM-Generated RTL Assertions with Quality-Aware Formal Verification," arXiv 2606.21451, June 2026.**
    URL: https://arxiv.org/pdf/2606.21451
    What: Mutation-guided refinement rejects weak or vacuous assertions, and surviving mutants are fed back for refinement (target kill rate of at least 85%).
    Relevance: **Overlaps the AQRS→ARS closed loop** (quality gap drives refinement). Theirs is functional, mutation-based, and LLM-driven. Ours is security-scoped, oracle/SPV-based, and template-driven.

26. **AssertLLM2: "A Comprehensive LLM Benchmark for Assertion Generation from Design Specifications," arXiv 2605.27472, 2026** (authors **unverified**).
    URL: https://arxiv.org/pdf/2605.27472
    What: Argues that syntax and FPV pass rates are insufficient because assertions can be trivial, vacuous, or weak.
    Relevance: Supports the motivation that "Proven" is not "good". Cite in §1.

---

## B. Hardware security verification metrics, properties, and assets

27. **S. Rekhi, K. Amberiadis, A. A. Akib, A. Srivastava, "Metrics and Methodology for Hardware Security Constructs," NIST CSWP 45, June 5, 2025.** DOI 10.6028/NIST.CSWP.45.
    URL: https://csrc.nist.gov/pubs/cswp/45/metrics-and-methodology-for-hardware-security-cons/final ; https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.45.pdf
    What: CWE/CAPEC-based *threat* (weaknesses per attack pattern) and *sensitivity* (attack patterns per weakness) metrics for hardware.
    Relevance: Hardware-native grounding for a severity term (CVSS_norm). Relevant if the severity term is kept.

28. **N. Farzana (Dipu), F. Farahmandi, M. Tehranipoor, "SoC Security Properties and Rules," IACR ePrint 2021/1014.**
    URL: https://eprint.iacr.org/2021/1014
    What: A public database of SoC security properties and rules mapped to vulnerabilities.
    Relevance: Security-property library. Contrast its property *collection* with our property *quality*.

29. **N. Farzana, A. Ayalasomayajula, F. Rahman, F. Farahmandi, M. Tehranipoor, "SAIF: Automated Asset Identification for Security Verification at the RTL," VTS 2021, pp. 1-7.**
    URL: https://ieeexplore.ieee.org/document/9441039
    What: Identifies RTL security assets via vulnerability analysis and adds metrics for assessing vulnerabilities of those assets.
    Relevance: Upstream asset identification. Alternative to NS31A-grounding of the SRTLC denominator.

30. **LAsset: "An LLM-assisted Security Asset Identification Framework for SoC Verification," DATE 2026, arXiv 2601.02624** (Univ. of Florida; author list **unverified** in result).
    URL: https://arxiv.org/abs/2601.02624v1 ; https://past.date-conference.com/proceedings-archive/2026/DATA/1373.pdf
    What: LLM agents identify primary and secondary assets from spec and RTL, refined against CWE. Up to 90% recall on SoCs. Releases NEORV32 (RISC-V) asset lists.
    Relevance: SRTLC denominator alternative. Also an evaluation path: compare NS31A-derived signals vs LAsset assets.

31. **A. Ayalasomayajula, N. Farzana, D. Pal, F. Farahmandi, "Prioritizing Information Flow Violations: Generation of Ranked Security Assertions for Hardware Designs," HOST 2024, pp. 128-138** (from DBLP; abstract not retrieved).
    URL: https://dblp1.uni-trier.de/pid/258/8478.html
    What: Generates security assertions from IFT violations and **ranks** them (details **unverified**, as the abstract was not retrieved).
    Relevance: **Novelty pressure on "security assertion ranking" (AQS).** Read the full text before claiming that AQS is the first security-assertion ranker.

32. **A. Meza, R. Kastner, "Information Flow Coverage Metrics for Hardware Security Verification," arXiv 2304.08263, 2023** (6 pages; venue **unverified**).
    URL: https://arxiv.org/abs/2304.08263v1
    What: "Hyperflow graphs" over RTL with IFT attributes. Security metrics measure flow *paths*, conditions, and rates for confidentiality, integrity, and availability, demonstrated on OpenTitan.
    Relevance: **Close to SAPC (path coverage into security signals) and SRTLC.** It is IFT-graph-based rather than SPV+COI, and has no Trojan-correlation validation. Must cite.

33. **W. Hu, A. Ardeshiricham, R. Kastner, "Hardware Information Flow Tracking" (survey).** Venue **unverified** (believed ACM CSUR 2021, not confirmed in this session).
    URL: https://par.nsf.gov/servlets/purl/10298114
    What: Taxonomy of hardware IFT by abstraction level and verification technique.
    Relevance: Background for SPV/taint, which is what SAPC uses.

34. **A. Ardeshiricham, W. Hu, J. Marxen, R. Kastner, "Register Transfer Level Information Flow Tracking for Provably Secure Hardware Design," DATE 2017.**
    URL: https://past.date-conference.com/proceedings-archive/2017/html/0765.html
    What: RTL-level IFT (RTLIFT), more than 5x faster than gate-level IFT.
    Relevance: Formal IFT background for SAPC.

35. **C. Deutschbein, A. Meza, F. Restuccia, R. Kastner, C. Sturton, "Isadora: Automated Information Flow Property Generation for Hardware Designs," ASHES 2021, arXiv 2106.07449.**
    URL: https://arxiv.org/abs/2106.07449v2
    What: IFT plus specification mining yields information-flow properties aligned with CWEs. Evaluated on a RISC-V core and on OpenTitan.
    Relevance: Security property generation with CWE alignment. Background for the CWE term.

36. **R. Zhang, N. Stanley, C. Griggs, A. Chi, C. Sturton, "Identifying Security Critical Properties for the Dynamic Verification of a Processor" (SCIFinder), ASPLOS 2017.**
    URL: https://www.cs.unc.edu/~csturton/papers/ASPLOS2017Zhang.pdf
    What: Uses errata plus ML to infer security-critical processor invariants (OR1200). Recovers 86.4% of manual properties.
    Relevance: Precedent for "security-critical signals defined by expert/errata properties", which is the same philosophy as the NS31A-grounded SRTLC denominator.

37. **R. Zhang, C. Sturton, "Transys: Leveraging Common Security Properties Across Hardware Designs," IEEE S&P 2020.**
    URL: https://par.nsf.gov/biblio/10207526 ; https://cps-vo.org/sites/cps-vo.org/files/cpsvo_file_nodes/Transys_Leveraging_Common_Security_Properties_Across_Hardware_Designs.pdf
    What: Translates security properties across designs (96% success, including RISC processors).
    Relevance: Upstream of our assertion source (cross-design translation). Context only.

38. **S. Imtiaz, U. Reinsalu, T. Ghasempouri, "Translating Common Security Assertions Across Processor Designs: A RISC-V Case Study," ISCAS 2025, arXiv 2502.10194.** DOI 10.1109/ISCAS56072.2025.11043977.
    URL: https://arxiv.org/abs/2502.10194
    What: Translates security assertions to RISC-V (Ibex). Nearly 100% translation across five security modules. Evaluated against LLM-generated Trojans.
    Relevance: **Self-citation candidate for the assertion source.** Note that it reports 5 modules, while CLAUDE.md claims 68 assertions over 9 modules. Confirm which paper is the actual "AVS" source (ISCAS 2025 vs the later AutoTrans preprint).

39. **Y. Lyu, P. Mishra, "System-on-Chip Security Assertions," arXiv 2001.06719, 2020** (venue **unverified**).
    URL: https://arxiv.org/abs/2001.06719
    What: Defines classes of security assertions for common SoC vulnerabilities. Security assertions catch inserted vulnerabilities that functional assertions miss.
    Relevance: Motivation that functional coverage is not security coverage.

40. **H. Witharana, A. Jayasena, A. Whigham, P. Mishra, "Automated Generation of Security Assertions for RTL Models," ACM journal (JETC per file name; venue and year **unverified**).**
    URL: https://www.cise.ufl.edu/research/cad/Publications/jetc22.pdf
    What: Automatic vulnerability analysis to generate security assertions across six vulnerability classes.
    Relevance: Security-assertion generation background.

41. **D. R. Ankireddy et al., "LASA: Enhancing SoC Security Verification with LLM-Aided Property Generation," arXiv 2506.17865, 2025.**
    URL: https://arxiv.org/abs/2506.17865
    What: Generates security SVAs with RAG, using commercial formal-tool *COI and proof-core coverage* in a feedback loop. Reports about 88% coverage and finds Hack@DAC'24 bugs.
    Relevance: **Uses COI/proof-core coverage as an assertion-quality feedback signal for security properties**, close to SRTLC. Differentiate on the expert-grounded security denominator and the Trojan validation.

42. **J. Rogers, N. Shakeel, D. Mankani, S. Espinosa, C. Chabra, K. Ryan, C. Sturton, "Security Properties for Open-Source Hardware Designs," arXiv 2412.08769, 2024.**
    URL: https://arxiv.org/abs/2412.08769
    What: Open SVA security properties for OR1200, PULPissimo, CVA6, and OpenPiton, tagged with CWE.
    Relevance: CWE-tagged security-property sets for RISC-V (CVA6). A possible external assertion set to test AQS generality.

43. **G. Dessouky et al., "HardFails: Insights into Software-Exploitable Hardware Bugs," USENIX Security 2019.**
    URL: https://www.usenix.org/conference/usenixsecurity19/presentation/dessouky
    What: Shows that state-of-the-art security verification (including formal) misses classes of RISC-V RTL bugs.
    Relevance: Motivation that "verified" does not mean secure, and that verification blind spots exist (SAPC/TCFC motivation).

44. **CRESS: "Quantifying Vulnerabilities of Attack Scenarios in Hardware Reverse Engineering," arXiv 2606.05459, 2026** (authors **unverified**).
    URL: https://arxiv.org/pdf/2606.05459
    What: Proposes a hardware-specific scoring system that is claimed to be more expressive than CVSS for hardware CWEs.
    Relevance: Evidence that CVSS is a poor fit for hardware. Supports dropping or replacing the CVSS term.

---

## C. Formal hardware Trojan detection

45. **M. Eslami, T. Ghasempouri, S. Pagliarini, "Reusing Verification Assertions as Security Checkers for Hardware Trojan Detection," ISQED 2022, pp. 1-6, arXiv 2201.01130.** DOI 10.1109/ISQED54688.2022.9806292.
    URL: https://arxiv.org/abs/2201.01130
    What: Defines a per-assertion **Security Coverage** metric from **JasperGold SPV** (nodes structurally covered by an assertion over all nodes) and an assertion-selection flow. Analyses 108 assertions across 35 OpenTitan IPs and evaluates on TrustHub.
    Relevance: **Biggest novelty threat to SAPC, and partially to SRTLC and AQS.** It is the same tool (SPV) with a per-assertion structural coverage that is already linked to Trojan detection. Co-author T. Ghasempouri is shared, so position SAPC as an extension of it.

46. **M. Eslami, T. Ghasempouri, S. Pagliarini, "SCARF: Securing Chips with a Robust Framework against Fabrication-time Hardware Trojans," IEEE Trans. Computers, 2024, arXiv 2402.12162.**
    URL: https://arxiv.org/abs/2402.12162
    What: Formalizes Security Coverage (covered vs vulnerable nodes, per assertion or per design) and adds online monitors during physical synthesis (up to +33.5% SC).
    Relevance: Second threat to SAPC/SRTLC. It is a journal-level definition of per-assertion security coverage.

47. **C. S. Chuah, C. Appold, T. Leinmüller, "Reliable Hardware Trojan Detection for RISC-V Processors using Formal Verification and Automation," ISQED 2026.**
    URL: https://isqed.org/English/Archives/2026/Technical_Sessions/79.html
    What: Model checking with automatically generated functionality plus **signal-connection properties**. Gives a complete property set detecting any Trojan that modifies the PC anywhere in a RISC-V core. Full proofs in under 24 h.
    Relevance: Direct comparison for AER and the Trojan evaluation stage (TES). **Caution:** the abstract does *not* mention free XOR variables, JasperGold, or Ibex. CLAUDE.md's claim that Chuah uses "free XOR variables (same mechanism as AER)" is **unverified** until the full paper is read.

48. **M. Qin, J. Li, J. Yan, Z. Hao, W. Hu, B. Liu, "HT-PGFV: Security-Aware Hardware Trojan Security Property Generation and Formal Security Verification Scheme," Electronics 13(21):4286, 2024.** The exact title is **unverified**; the topic is confirmed.
    URL: https://www.mdpi.com/2079-9292/13/21/4286/htm
    What: Feature matching, then COI-based Trojan path location, then IFT trace mining, producing Trojan SVA properties that are verified with IFT.
    Relevance: Uses **cone-of-influence analysis** for Trojan paths (relevant to SRTLC/SAPC). Formal Trojan-property baseline.

49. **M. R. Heidari Iman, S. Ahmadi-Pour, R. Drechsler, T. Ghasempouri, "Processor Vulnerability Detection with the Aid of Assertions: RISC-V Case Study," NorCAS 2024** (venue from CLAUDE.md and the hosting URL; the PDF confirms a 2024 IEEE paper).
    URL: https://agra.informatik.uni-bremen.de/doc/konf/norcas2024_sap.pdf
    What: Automatically mines security assertions for a RISC-V processor and detects all inserted Trojans.
    Relevance: Same group, same processor family. Shows that detection-rate-only evaluation is the status quo, which AQS improves upon.

50. **A. Nahiyan, M. Sadi, R. Vittal, G. Contreras, D. Forte, M. Tehranipoor, "Hardware Trojan Detection through Information Flow Security Verification," ITC 2017, arXiv 1803.04102.** The author list is from memory of the paper and is **unverified** in the snippet; title and venue are verified.
    URL: https://arxiv.org/abs/1803.04102v1
    What: Detects Trojans as information-flow security policy violations, using ATPG to propagate assets. Notes that **JasperGold SPV missed AES-T2000/T2100** because the analyst must know which internal signal to check.
    Relevance: **Documents an SPV limitation.** SAPC must acknowledge that SPV path existence depends on the chosen source/target signals.

51. **J. Rajendran, V. Vedula, R. Karri, "Detecting Malicious Modifications of Data in Third-Party Intellectual Property Cores," DAC 2015.** Title and authors are **unverified**; the DOI 10.1145/2744769.2744823 abstract matches (formal BMC properties for corruption of critical registers).
    URL: https://www.doi.org/10.1145/2744769.2744823
    What: BMC-based properties detect unauthorized corruption of critical data in 3PIPs.
    Relevance: Early property-based Trojan detection on critical registers. Ancestor of AER's integrity checks.

52. **J. Rajendran, A. M. Dhandayuthapany, V. Vedula, R. Karri, "Formal Security Verification of Third Party Intellectual Property Cores for Information Leakage," VLSID 2016** (venue **unverified**).
    URL: https://cyber.nyu.edu/2016/01/04/formal-security-verification-third-party-intellectual-property-cores-information-leakage-2/
    What: Formal verification of 3PIPs for unauthorized leakage, validated on TrustHub.
    Relevance: Leak-category formal Trojan detection (TCFC "Leak").

53. **N. Fern, I. San, K.-T. Cheng, "Detecting Hardware Trojans in Unspecified Functionality Through Solving Satisfiability Problems," ASP-DAC 2017.**
    URL: https://www.aspdac.com/aspdac2017/archive/pdf/7B-2_add_file.pdf
    What: Trojans hidden in *unspecified* signal behavior evade verification. Detection is formulated as SAT/SMT over attacker-controllable "don't-care" values.
    Relevance: **Close relative of the AER oracle.** A free variable over unspecified behavior asks whether a value can leak or corrupt unnoticed. Must cite next to AER.

54. **A. L. Duque Antón, J. Müller, L. Deutschmann, M. R. Fadiheh, D. Stoffel, W. Kunz, "A Golden-Free Formal Method for Trojan Detection in Non-Interfering Accelerators," DATE 2024 (per DBLP listing; venue **unverified**), arXiv 2312.06515.**
    URL: https://arxiv.org/abs/2312.06515
    What: 2-safety property checking guarantees exhaustive detection of any sequential Trojan regardless of payload. Detects all TrustHub accelerator Trojans.
    Relevance: Payload-independent formal guarantee. A stronger completeness notion than AER, but limited to accelerators.

55. **M. R. Fadiheh, D. Stoffel, C. Barrett, S. Mitra, W. Kunz, "Processor Hardware Security Vulnerabilities and their Detection by Unique Program Execution Checking," DATE 2019, pp. 994-999.**
    URL: https://past.date-conference.com/proceedings-archive/2019/html/1082.html
    What: UPEC is an exhaustive formal (2-safety) method for covert channels in in-order processors.
    Relevance: Covert-channel category (TCFC). Formal processor security without hand-written assertions.

56. **A. Waksman, M. Suozzo, S. Sethumadhavan, "FANCI: Identification of Stealthy Malicious Logic Using Boolean Functional Analysis," ACM CCS 2013, pp. 697-708.**
    URL: https://api.openalex.org/works/doi:10.1145%2F2508859.2516654
    What: Flags nearly-unused logic (low control value) as Trojan candidates.
    Relevance: Classic static Trojan detection baseline (related work).

57. **J. Zhang, F. Yuan, L. Wei, Z. Sun, Q. Xu, "VeriTrust: Verification for Hardware Trust," DAC 2013 (journal: IEEE TCAD 34(7), 2015).**
    URL: https://webarchive.lib.cuhk.edu.hk/20210324224318mp_/http:/www.cse.cuhk.edu.hk/~qxu/zhang-dac13.pdf
    What: Flags trigger inputs that are never exercised by verification corners. DeTrust (CCS 2014) shows how to evade it and FANCI.
    Relevance: Verification-based Trojan detection baseline. DeTrust motivates evasion-resistance (AER) thinking.

58. **L.-E. Ploix, A. Armstrong, T. Melham, R. Lin, H. Wang, A. Courtney, "Comprehensive Formal Verification of Observational Correctness for the CHERIoT-Ibex Processor," arXiv 2502.04738, 2025.**
    URL: https://arxiv.org/abs/2502.04738
    What: Unbounded Jasper proofs of Ibex-derived core equivalence against the Sail RISC-V spec. Adapted by lowRISC to plain Ibex.
    Relevance: State-of-the-art formal on Ibex. Shows the Ibex + JasperGold tractability of the setup.

59. **"Validation of Hardware Security and Trust: A Survey," arXiv 1801.00649** (authors **unverified**; likely the Mishra/Farahmandi group).
    URL: https://arxiv.org/pdf/1801.00649
    What: A survey of simulation, formal, and hybrid trust-validation methods.
    Relevance: Survey citation for §2.

---

## D. Hardware Trojan benchmarks and generation

60. **B. Shakya, T. He, H. Salmani, D. Forte, S. Bhunia, M. Tehranipoor, "Benchmarking of Hardware Trojans and Maliciously Affected Circuits," J. Hardware and Systems Security 1(1):85-102, 2017.** DOI 10.1007/s41635-017-0001-6.
    URL: https://tehranipoor.ece.ufl.edu/wp-content/uploads/2021/07/2017-HaSS-Trohanbanchmark.pdf
    What: The TrustHub trust-benchmark suite and vulnerability analysis flow.
    Relevance: **Primary TrustHub citation for BENCH.**

61. **M. Tehranipoor, F. Koushanfar, "A Survey of Hardware Trojan Taxonomy and Detection," IEEE Design & Test of Computers, 2010.**
    URL: https://aceslab.org/w/Tehranipoor2010a
    What: The Trojan taxonomy by physical, activation, and action characteristics.
    Relevance: Taxonomy grounding for the 6 categories used in BENCH/TCFC.

62. **J. Cruz, Y. Huang, P. Mishra, S. Bhunia, "An Automated Configurable Trojan Insertion Framework for Dynamic Trust Benchmarks," DATE 2018.**
    URL: https://past.date-conference.com/proceedings-archive/2018/html/0176.html
    What: Configurable netlist Trojan insertion (TRIT-style benchmark sets on TrustHub).
    Relevance: Benchmark-generation precedent.

63. **A. Sarihi, P. Jamieson, A. Patooghy, A.-H. A. Badawy, "TrojanForge: Generating Adversarial Hardware Trojan Examples Using Reinforcement Learning," arXiv 2405.15184, 2024.**
    URL: https://arxiv.org/abs/2405.15184
    What: A GAN-like RL inserter versus detectors. Its Trojans evade four detectors on ISCAS-85.
    Relevance: Argues that TrustHub is small and biased. Supports building our own processor benchmark.

64. **J. Bhandari, R. Sadhukhan, P. Krishnamurthy, F. Khorrami, R. Karri, "SENTAUR: Security EnhaNced Trojan Assessment Using LLMs Against Undesirable Revisions," arXiv 2407.12352, 2024.**
    URL: https://arxiv.org/abs/2407.12352
    What: LLM-generated RTL Trojans from spec plus Trojan-effect descriptions.
    Relevance: LLM-based RTL Trojan generation (BENCH construction context).

65. **M. O. Faruque, P. Jamieson, A. Patooghy, A.-H. A. Badawy, "Unleashing GHOST: An LLM-Powered Framework for Automated Hardware Trojan Design," arXiv 2412.02816, 2024.** First-author initials are **unverified**.
    URL: https://arxiv.org/abs/2412.02816v1
    What: GPT-4/Gemini/Llama insert Trojans into SRAM, AES, and UART. 88.9% synthesizable. 14 public benchmarks.
    Relevance: LLM-Trojan benchmark precedent.

66. **TrojanGYM: "A Detector-in-the-Loop LLM for Adaptive RTL Hardware Trojan Insertion," arXiv 2601.17178, Jan 2026** (NYU; authors **unverified**).
    URL: https://arxiv.org/abs/2601.17178v1
    What: Multi-agent LLM Trojan insertion with GNN-detector feedback. Up to 83% evasion.
    Relevance: Shows that TrustHub-style benchmarks hide detector weaknesses. Related to adversarial robustness (AER motivation).

67. **G. Kokolakis, A. Moschos, A. D. Keromytis, "Harnessing the Power of General-Purpose LLMs in Hardware Trojan Design," AIHWS 2024 (ACNS workshop).**
    URL: https://coeus.ece.gatech.edu/2024/03/30/AIHWS2024
    What: LLM-assisted Trojan insertion into a **RISC-V CPU**, targeting integrity and availability, validated on an FPGA.
    Relevance: RISC-V processor Trojan generation precedent for BENCH.

68. **A. Damljanovic, A. Ruospo, E. Sánchez, G. Squillero, "A Benchmark Suite of RT-level Hardware Trojans for Pipelined Microprocessor Cores," DDECS 2021, pp. 51-56.**
    URL: https://iris.polito.it/handle/11583/2882631
    What: RT-level Trojans in a RISC pipelined core, with area, power, and frequency impact.
    Relevance: **Closest prior processor-RTL Trojan benchmark.** Must cite for BENCH novelty.

69. **"Work-in-Progress: RISC-V-Dilemma-1: A Hardware Trojan Benchmark Based on Seeker's Dilemma Approach," SAT-CPS 2025 (ACM)** (authors and details **unverified**; listing only).
    URL: https://discovery.fiu.edu/display/pub554918
    What: A RISC-V Trojan benchmark (WiP).
    Relevance: **Direct competitor for the "RISC-V Trojan benchmark" claim.** Read before claiming novelty.

70. **TrojanLoC / TrojanInS: "LLM-based Framework for RTL Trojan Localization," arXiv 2512.00591, 2025** (authors **unverified**).
    URL: https://arxiv.org/pdf/2512.00591
    What: More than 16k Trojaned Verilog designs with line-level labels.
    Relevance: Large-scale RTL Trojan dataset (not RISC-V-specific).

71. **A. Sarihi et al., "Trojan Playground: A Reinforcement Learning Framework for Hardware Trojan Insertion and Detection," arXiv 2305.09592, 2023** (author list **unverified**).
    URL: https://arxiv.org/pdf/2305.09592
    What: RL-based Trojan insertion and detection benchmark generation.
    Relevance: Benchmark-generation context.

72. **Hayashi & Ruggiero, IEEE Access, Jan 2025** (title **unverified**).
    URL: https://doaj.org/article/4ea3db60629b4fb2ad08a04f941d34fa
    What: A 3,808-instance dataset combining TrustHub, ISCAS, RISC-V, and MIPS for ML/LLM Trojan detection.
    Relevance: Shows that RISC-V Trojan instances already exist in datasets, which limits BENCH novelty claims.

---

## (i) Ranked top-15 must-cite list (6-page paper)

| # | Paper | Why it is unavoidable |
|---|---|---|
| 1 | Eslami, Ghasempouri, Pagliarini, ISQED 2022 [45] | SPV-based per-assertion Security Coverage linked to Trojans. Direct predecessor of SAPC. |
| 2 | Eslami et al., SCARF, IEEE TC 2024 [46] | Journal definition of per-assertion security coverage. |
| 3 | Chuah, Appold, Leinmüller, ISQED 2026 [47] | Formal RISC-V Trojan detection with completeness claim. Main comparison. |
| 4 | Imtiaz, Reinsalu, Ghasempouri, ISCAS 2025 [38] | Assertion source (self-cite). |
| 5 | Chockler, Kupferman, Kurshan, Vardi, CAV 2001 [2] | Mutation-based coverage in model checking. AER's formal ancestor. |
| 6 | Kupferman, CONCUR 2006 [7] | Vacuity + coverage as "sanity checks" after a proof. Frames the whole paper. |
| 7 | Fern, San, Cheng, ASP-DAC 2017 [53] | Free-value/unspecified-functionality Trojans via SAT. Closest to the AER oracle. |
| 8 | Meza & Kastner, 2023 [32] | Information-flow *path* coverage metrics for HW security. Close to SAPC. |
| 9 | HierSVA, 2026 [24] | Multi-axis assertion quality incl. formal-core + mutation coverage. Overlaps SRTLC/AER (functional). |
| 10 | Ayalasomayajula et al., HOST 2024 [31] | Ranked security assertions. Prior art for "ranking". |
| 11 | Shakya et al., JHSS 2017 [60] | TrustHub benchmark. |
| 12 | Damljanovic et al., DDECS 2021 [68] | Prior processor RTL Trojan benchmark. |
| 13 | HT-PGFV, Electronics 2024 [48] | COI-based formal Trojan property generation. |
| 14 | Nahiyan et al., ITC 2017 [50] | IFS-based Trojan detection plus documented SPV limitation. |
| 15 | Inozemtseva & Holmes, ICSE 2014 [18] | Canonical caution on coverage-vs-effectiveness correlation. Pre-empts the reviewer critique of the Pearson claim. |

Alternates if space allows: Cadence proof-core patent [10] (justify COI vs proof core), LASA [41], Krishnamurthy et al. 2026 [25] (closed loop), NIST CSWP 45 [27] (only if a severity term stays), Iman et al. NorCAS 2024 [49], Fadiheh UPEC [55].

---

## (ii) Gaps in the literature this paper can honestly claim

Avoid "first" claims for any single metric. Defensible claims are about the **combination** and the **validation**:

1. **Security-scoped COI coverage with an expert-grounded denominator.** COI and proof-core coverage are standard tool features [10], and SCARF/Eslami [45,46] already compute node coverage per assertion. What appears new is a denominator defined by *expert-written security properties from a reference processor* (NS31A) rather than all nodes, CWE heuristics, or LLM-identified assets [29,30]. The defensible claim is "a security-scoped coverage", not "a new coverage metric". Expect a reviewer question on COI vs proof core.

2. **Per-assertion behavioral evasion resistance as a ranking term.** Mutation coverage in model checking [2,3,7], unspecified-functionality SAT detection [53], and mutation-kill quality scores for LLM assertions [24,25] all exist. None that we found turns a *free-variable perturbation of security-critical signals* into a per-assertion score used to rank *security* assertions. The defensible claim is "per-assertion, security-signal-targeted application of formal mutation coverage".

3. **Joint behavioral (FPV oracle) and structural (SPV) evidence per assertion.** SPV-based structural coverage exists [45,46], and IFT path metrics exist [32]. We found no work that combines SPV structural reachability with an FPV oracle check per assertion and shows that the two disagree (AER=1 but SAPC<100%). This "behavioral vs structural" disagreement is likely the cleanest novel insight, provided it is demonstrated empirically.

4. **Validating a pre-Trojan formal quality score against an independent Trojan benchmark, by correlation, on a RISC-V core.** Eslami et al. [45] evaluated on TrustHub, and HierSVA [24] correlates assertion quality with fault detection on functional designs. We did not find a paper that computes a formal assertion-quality score *before* any Trojan exposure and then reports its rank correlation with detection over a processor-level Trojan set across multiple security modules. This is the strongest contribution claim, but it rests entirely on the statistical result. Report Spearman/Kendall plus Pearson, give n and confidence intervals, and control for assertion count and module size [18].

5. **Ibex security-module Trojan benchmark.** Processor RTL Trojan suites [68,69,72] and LLM-generated RISC-V Trojans [38,67] already exist. A claim of "a TrustHub-category-structured Trojan set over 9 Ibex security modules, independent of the assertion set" is defensible. A claim of "first RISC-V Trojan benchmark" is not.

**Things to fix in CLAUDE.md or the paper draft before writing:**
- The Chuah ISQED 2026 abstract does not mention free XOR variables, JasperGold, or Ibex. Do not state "same mechanism as our AER" until the full paper confirms it.
- The ISCAS 2025 self-citation reports 5 security modules. Reconcile this with the "68 assertions, 9 modules" claim, or identify the correct source paper.
- "AutoAssert" collides with DATE 2026 AutoAssert/TrustAssert [23] and with AutoAssert 1 (arXiv 2508.07371). This supports renaming.
