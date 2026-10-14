# New Metric Ideas: Research Scout Report

Date: 2026-10-14. Scope: candidate 4th metric, and a graded version of AER, for the AQS ranker (SRTLC, AER, SAPC). The target is IEEE ETS 2027 (6 pages).

**Citation policy.** Every reference below came from a web search or fetch run in this session, and the URL was returned by the tool. Where I only saw a search snippet and could not confirm the content, the entry is marked "(snippet only)". Anything I could not confirm at all is marked **unverified**.

---

## 0. Findings that affect the existing paper (read first)

1. **The Chuah ISQED 2026 abstract does not mention free XOR variables or JasperGold.** I fetched the ISQED 2026 session page: https://isqed.org/English/Archives/2026/Technical_Sessions/79.html. It covers model checking, signal-connection properties, a complete property set for Trojans that modify the PC, and an automation tool. CLAUDE.md says Chuah uses "free XOR variables (same mechanism as our AER oracle)". That claim is **unverified** from the abstract. Check it against the full paper before the paper says "extends Chuah's free-variable model". The same group also published "Formal Verification of Security Properties on RISC-V Processors" (MBMV 2024): https://www.vde-verlag.de/proceedings-en/456267021.html. Per the search snippet, they used Jasper on a commercial RISC-V core.
2. **Co-author Tara Ghasempouri wrote the closest SAPC prior art.** arXiv 2201.01130 is Eslami, Ghasempouri, Pagliarini, ISQED 2022: https://arxiv.org/abs/2201.01130 (fetched). It is self-related prior work, so cite it openly as "our group's earlier SPV-based Security Coverage". SCARF reuses that metric: https://arxiv.org/pdf/2402.12162.
3. **arXiv 2502.10194** is "Translating Common Security Assertions Across Processor Designs: A RISC-V Case Study" (https://arxiv.org/pdf/2502.10194). It covers Ibex, 100% of modules, and a "Trojan Power Index". It looks like the Project A translation paper behind the `[CITATION NEEDED]` in CLAUDE.md. Confirm with Sharjeel.
4. **Mutation kill rate stays rejected.** https://arxiv.org/pdf/2606.21451 ("Closing the Loop on LLM-Generated RTL Assertions with Quality-Aware Formal Verification") and https://arxiv.org/pdf/2608.12635 (GateTruth) both appeared in search results.

---

## 1. Search log (21 searches + 4 fetches)

The searches covered these topics:
- security assertion coverage metrics, 2024-2025
- information-flow coverage and non-interference
- rare-node Trojan triggers
- detection latency
- SCOAP and security
- assertion redundancy and minimization
- Chuah ISQED 2026
- JasperGold proof-core and COI coverage
- bounded proof depth and time bombs
- reset-state security (CWE-1271)
- k-bit fault injection robustness
- privilege-mode coverage on RISC-V
- taint-propagation latency
- security property completeness, 2025-2026
- minimum-fault attack resilience
- Hoskote formal coverage
- shortest-CEX and tamper-duration stealth
- hyperproperty and self-composition
- GateTruth and 2606.21451
- Trojan trigger rarity, 2024-2025
- assertion strength and subsumption
- SPV taint and sequential depth
- Ibex security benchmarks
- multi-cycle fault duration
- NICOLE (DATE 2019)

### Key prior work found (verified URLs)

| Topic | Work | URL |
|---|---|---|
| SPV assertion security coverage | Eslami, Ghasempouri, Pagliarini, ISQED 2022 | https://arxiv.org/abs/2201.01130 |
| Assertion-set completeness to expose Trojans and blindspots | Fern & Cheng, DATE 2019 | https://past.date-conference.com/proceedings-archive/2019/html/0322.html |
| Information-flow coverage (flow paths, conditions, rates) | Meza & Kastner, arXiv 2023 (fetched) | https://arxiv.org/abs/2304.08263 |
| System-level IFT security coverage (availability) | Coşkun et al., ASP-DAC 2024 / MBMV 2024 (snippet only) | https://curat.informatik.uni-bremen.de/doc/konf/aspdac2024_ec.pdf |
| Quantitative IF (vs strict non-interference) | QFlow | https://arxiv.org/pdf/2109.02379 |
| Joint IF quantification | Kastner et al., DATE 2016 (snippet only) | https://past.date-conference.com/proceedings-archive/2016/pdf/1008.pdf |
| Coverage for security property verification (PSS stimuli) | MBMV 2025 (snippet only) | https://www.vde-verlag.de/proceedings-en/456515019.html |
| k-fault resistant partitioning (OpenTitan lockstep, single bit-flip) | ePrint 2024/247 (snippet only) | https://eprint.iacr.org/2024/247 |
| SAT-based fault-resistance verification | FIRMER, TCHES 2024 | https://tches.iacr.org/index.php/TCHES/article/view/11782 |
| Symbolic fault injection | FIVER, TCHES 2021 | https://tches.iacr.org/index.php/TCHES/article/view/9072 |
| Rarity metric (min(P(0),P(1)) from simulation) | COVERT, DATE 2026 / arXiv | https://arxiv.org/abs/2601.11939v1 |
| Rare branches = Trojan coverage | Dai group, ITC 2018 | https://www.cise.ufl.edu/research/cad/Publications/itc18.pdf |
| Rare-node mining at SoC scale (SCOAP-based) | arXiv 2511.08703 | https://arxiv.org/html/2511.08703v1 |
| SCOAP features for Trojan-net classification | (snippet only) | https://archive.cps-vo.org/node/97496 |
| BMC misses late-trigger Trojans | arXiv 1803.04102 | https://arxiv.org/pdf/1803.04102 |
| Golden-free 2-safety Trojan detection (unbounded triggers) | arXiv 2312.06515 | https://arxiv.org/pdf/2312.06515 |
| Assertion reduction preserving coverage | Arcane, arXiv 2605.10107 | https://arxiv.org/pdf/2605.10107 |
| Assertion redundancy via subsumption (review) | arXiv 2607.07444 | https://arxiv.org/pdf/2607.07444 |
| Assertion figure of merit / ranking | Illinois (snippet only) | https://experts.illinois.edu/en/publications/a-figure-of-merit-for-assertions-in-verification/ |
| RTL code-coverage for assertions | Athavale et al., UIUC (snippet only) | https://www.ideals.illinois.edu/items/100288 |
| JasperGold bounded proof coverage ("K cycles from reset") | SemiWiki COV app article | https://semiwiki.com/eda/3085-jaspergold-cov-app-the-swiss-army-knife-for-verification/ |
| CWE-1271 (uninitialised security regs on reset) | MITRE | https://cwe.mitre.org/data/definitions/1271 |
| Privilege-mode coverage (simulation-side) | Imperas (snippet only) | https://www.imperas.com/node/1875 |
| Formal verification of RISC-V PMP | arXiv 2211.02179 | https://arxiv.org/pdf/2211.02179 |
| Security-property benchmark (120 SVA, Trust-Hub) | Rogers et al. 2025 (snippet only) | https://www.cs.unc.edu/~csturton/papers/rogers2025hardware.pdf |
| Self-composition for non-interference | SecIC3, arXiv 2601.21353 (snippet only) | https://www.alphaxiv.org/abs/2601.21353 |

**Gaps found in the literature** (useful for novelty claims):
- No paper found that measures **assertion detection latency** (cycles from security violation to assertion firing) as a quality metric.
- No paper found that grades assertions by the **minimum tamper budget** (bits × cycles) an evading attacker needs. Fault-resistance work (FIRMER, ePrint 2024/247) uses k-fault models, but for cryptographic countermeasures, not for ranking assertions.
- No paper found on **privilege-mode or reset-context coverage of assertions under formal verification**. Imperas does privilege coverage for simulation stimuli only.
- No paper found that uses **formal reachability depth as a rarity proxy** for coverage weighting. COVERT and ITC'18 rarity are simulation-based.

---

## 2. Candidate metrics

Shared building block: the AER oracle harness already has, per assertion A:
- a free variable `fv` XOR-injected into security signal s;
- a predicate `viol` (the intended security property is broken);
- a signal `fail_A` (assertion A fires).

`fail_A` must be an explicit checker signal, not just the `assert` directive. AER already needs this to express "violated while silent". Candidates C1-C3 reuse this harness, so they are mostly extra `cover`/`assert` directives in the same 12 AER runs.

### C1. Evasion Budget (EB): graded AER (recommended, replaces binary AER)

- **Intuition:** a good assertion forces the attacker to tamper with more bits for more cycles before it can slip through.
- **Formula:** define a budget lattice B = {(k, d)}, with k = max flipped bits per cycle (Hamming weight of `fv`) in {1, 2, W} and d = number of tampered cycles in {1, 2, 4, 8, unbounded}. Order the lattice by cost c = k·d (bit-cycles); for d = unbounded, use the bound reached. Let L be the number of levels and ℓ*(A) the lowest level at which the evasion cover is reachable. Then:
  `AER_g(A) = 1` if no level is reachable (fully Proven, the same as binary AER = 1), otherwise `AER_g(A) = (ℓ*(A) − 1) / L`.
  An alternative in [0,1] is `log2(1 + c_min) / log2(1 + c_max)`.
- **JasperGold:** in the AER TCL, add rigid free variables `k_lim` and `d_lim` with `assume property ($stable(k_lim) && $stable(d_lim))`. Add `assume property ($countones(fv) <= k_lim)` and a counter `tcnt` that increments when `fv != 0`, with `assume property (tcnt <= d_lim)`. Then add one `cover property (viol && !fail_A && k_lim==K && d_lim==D)` per lattice point. An unreachable cover means A resists that budget. JG's minimal-length witness also gives an "evasion depth" for free. Covers 0 extra runs; the 12 AER runs just get bigger (~12-15 covers per assertion).
- **Gap closed:** binary AER cannot tell a 1-bit, 1-cycle bypass from one that needs a full-width persistent tamper. That collapses the ranking, because most proven or most failed assertions tie. A graded score gives AQS variance, which the Pearson correlation needs.
- **Closest prior work:** k-fault resistance, https://eprint.iacr.org/2024/247 and https://tches.iacr.org/index.php/TCHES/article/view/11782 (fault models with k faults, applied to countermeasures). Chuah ISQED 2026 (binary, PC-level).
- **Novelty verdict:** likely novel as a per-assertion ranking score. The k-fault concept is borrowed (cite it).
- **Risk:** low to medium. Rigid free variables can slow proofs, so fall back to separate tasks per (k, d) if needed. For combinational PMP, d is meaningless; use k only.

### C2. Detection Latency (DLat)

- **Intuition:** an assertion that fires 6 cycles after the violation gives the payload a window to finish, for example leaking a CSR value. Earlier detection is better.
- **Formula:** `DLat(A) = min N such that  viol |-> ##[0:N] fail_A  is Proven` (under the AER oracle). Score: `DLat_n(A) = 1 − min(DLat(A), N_max)/N_max`, with N_max = 8 for example. If the assertion is never forced to fire (AER CEX), DLat_n = 0.
- **JasperGold:** in the AER run, add `cover property (viol ##1 (!fail_A)[*N])` for N = 1..N_max. The largest reachable N + 1 gives the latency. 0 extra runs.
- **Gap closed:** AER and EB ask whether the assertion fires at all; DLat asks how fast. SRTLC and SAPC are timing-blind.
- **Closest prior work:** none found. The search for "detection latency" assertion metrics returned only kill-rate and assertion-density work, e.g. https://arxiv.org/pdf/2606.21451. Fern & Cheng (DATE 2019) is related only conceptually.
- **Novelty verdict:** likely novel.
- **Risk:** medium. It is trivially 0 for PMP (combinational) and for single-cycle immediate checks, so variance may be low. Many Ibex security assertions are `|=> ` one-cycle checks, so DLat may be in {0, 1} for most of them. Run a pilot on 2 modules before committing.

### C3. Privilege/Reset Context Coverage (PRCC) (recommended as 4th metric candidate)

- **Intuition:** an assertion that is only non-vacuous in M-mode is blind to U-mode attacks, and one that only activates after the core is fully running misses the reset window (CWE-1271). AVS "non-vacuous" is a single global check; it does not say in which contexts the assertion actually guards anything.
- **Formula:** contexts C = {U, M, Debug} × {post-reset window (first R cycles after `rst_ni` rises), steady state}, so |C| = 6 (fewer for modules where Debug is unreachable). Let C_reach(M) be the contexts reachable in module M (design-only covers). Then:
  `PRCC(A) = |{c ∈ C_reach(M) : cover(antecedent_A ∧ c) reachable}| / |C_reach(M)|`.
  A stronger variant uses the EB/AER evasion cover restricted to each context. That counts contexts where A is evasion-resistant, not only active.
- **JasperGold:** one extra FPV run per bind file (10 runs, cover-only, cheap). Per assertion, add 6 `cover property` directives on `antecedent && priv_lvl == PRIV_LVL_{U,M} / debug_mode` and `antecedent && (rst_cnt < R)`. Ibex has `priv_mode_id`/`priv_lvl_q` in CSR and `debug_mode` in the controller; PMP takes `priv_mode_i` directly. Python takes the ratio from the JG cover report.
- **Gap closed:** SRTLC (which lines), SAPC (which input paths) and AER (which values) are all context-agnostic. PRCC adds the RISC-V-specific dimension of privilege level, which is exactly where the 9 security modules differ. It also maps onto the TrustHub "Privilege" category without the expensive 42-run TCFC.
- **Closest prior work:** Imperas privilege-mode coverage for simulation (https://www.imperas.com/node/1875, snippet only). CWE-1271 (https://cwe.mitre.org/data/definitions/1271). Formal PMP verification (https://arxiv.org/pdf/2211.02179) checks correctness, not assertion coverage. Coverage estimation for model checking (Hoskote DAC'99, DOI 10.1145/309847.309936, metadata only).
- **Novelty verdict:** likely novel as a formal per-assertion metric. The idea of privilege-mode coverage exists in simulation, so word the claim carefully.
- **Risk:** low cost, but there is a saturation risk: many assertions may reach all contexts, giving PRCC = 1 everywhere. Pilot on CSR and DO first. The antecedent must be extractable as a named sequence; this is easy if the bind files use `property ... a |-> b`.

### C4. Rarity-Weighted Security Coverage (RWSC)

- **Intuition:** Trojans hide behind rare triggers, so covering rarely reachable security logic should count more than covering always-active logic.
- **Formula:** for each security-critical branch or line l, rarity weight `r_l = depth_l / max depth`, where depth_l is the minimal JG cover-witness depth for "branch l taken" from reset. Then `RWSC_M = Σ_{l ∈ ∪COI} r_l / Σ_l r_l`.
- **JasperGold:** auto-generate `cover` on each branch condition of the security-critical lines, which is about 10 runs (one per module). Python reweights the SRTLC numerator and denominator.
- **Gap closed:** SRTLC treats all lines equally.
- **Closest prior work:** COVERT rarity (https://arxiv.org/abs/2601.11939v1), ITC'18 rare branches (https://www.cise.ufl.edu/research/cad/Publications/itc18.pdf), SCOAP rare-node mining (https://arxiv.org/html/2511.08703v1).
- **Novelty verdict:** partial overlap. Rarity-weighted coverage is known; using formal witness depth as the rarity estimate is new, but reviewers may say depth ≠ activation probability.
- **Risk:** medium-high. It is a reweighting of SRTLC, not an independent dimension, and adds a third coverage-flavoured term. Not recommended.

### C5. Unique Contribution / Redundancy (UCR)

- **Intuition:** an assertion whose COI and paths are fully subsumed by others adds no security value to the set.
- **Formula:** leave-one-out: `UCR(A) = [SRTLC(S) − SRTLC(S∖A)] + [SAPC(S) − SAPC(S∖A)]`, normalised. Optionally, a JG implication check `A_j → A_i` flags strict subsumption.
- **Cost:** 0 runs (Python on existing COI and SPV data). Subsumption checks would be up to n² cheap JG proofs per module.
- **Closest prior work:** Arcane (https://arxiv.org/pdf/2605.10107) and the subsumption discussion in https://arxiv.org/pdf/2607.07444. CLAUDE.md already defines `SRTLC_contrib` as a unique COI contribution.
- **Novelty verdict:** partial overlap, low novelty. It is already implicit in SRTLC_contrib.
- **Risk:** low cost, but adds little. Use it as a tie-breaker or in a sentence, not as a metric.

### C6. Information-Flow Coverage via SPV non-interference (IFC)

- **Intuition:** per assertion, count security sources (e.g. CSR secret or PMP config) whose taint can reach a sink without passing through any signal in the assertion's COI.
- **Cost:** ~12 SPV runs with blocking (`stopat`/waivers on COI nets).
- **Closest prior work:** Meza & Kastner hyperflow coverage (https://arxiv.org/abs/2304.08263), Eslami/Ghasempouri SPV coverage (https://arxiv.org/abs/2201.01130), QFlow (https://arxiv.org/pdf/2109.02379), ASP-DAC 2024 IFT coverage.
- **Novelty verdict:** pre-empted or heavily overlapping. It also overlaps SAPC itself.
- **Risk:** high (reviewer novelty pushback, SPV run cost). Not recommended.

---

## 3. Summary table

| # | Metric | Extra JG runs | Independent of SRTLC/AER/SAPC? | Novelty | Risk | Verdict |
|---|---|---|---|---|---|---|
| C1 | EB, graded AER | 0 (bigger AER runs) | refines AER | likely novel (borrowing k-fault) | low-med | **Adopt as AER grading** |
| C2 | DLat, detection latency | 0 (in AER runs) | yes (temporal) | likely novel | med (low variance) | Optional; pilot first |
| C3 | PRCC, privilege/reset context coverage | ~10 cover runs | yes (context) | likely novel (formal, per-assertion) | low cost, saturation risk | **Best 4th-metric candidate** |
| C4 | RWSC, rarity-weighted coverage | ~10 | no (reweights SRTLC) | partial overlap | med-high | Reject |
| C5 | UCR, unique contribution | 0 | no | partial overlap / low | low | Tie-breaker only |
| C6 | IFC, SPV non-interference | ~12 SPV | no (overlaps SAPC) | pre-empted | high | Reject |

---

## 4. Recommendation

1. **Make AER graded via the Evasion Budget (C1).** It is not a new metric, so the advisor-approved metric count stays at 3. It costs zero extra runs, fixes binary AER's tie problem (which otherwise weakens the AQS-vs-detection Pearson correlation), and borrows a well-cited k-fault model. Optionally, report the JG minimal-witness length (evasion depth) and DLat (C2) from the same runs as descriptive columns, not AQS terms.
2. **At most one new metric: PRCC (C3), and only if a 2-module pilot (CSR, DO) shows variance.** If most assertions score PRCC = 1, drop it and **stay at 3 metrics**. The 6-page limit favours 3 well-explained metrics plus a graded AER over 4 thin ones.
3. Do not add RWSC, UCR or IFC as metrics: they overlap SRTLC/SAPC or are pre-empted (Meza & Kastner 2023; Eslami/Ghasempouri 2022).

### Suggested 4-week plan

- **Week 1:** extend the AER oracle template with `k_lim`/`d_lim` covers and pilot on CSR + DO; add the PRCC covers in the same pilot.
- **Week 2:** run all 12 AER runs with EB, then decide on PRCC.
- **Week 3:** run PRCC (10 cover runs) if kept.
- **Week 4:** Python aggregation (`compute_aer.py` gains `aer_graded`), then sensitivity of the Pearson r to binary vs graded AER, which is itself a nice ablation for §7c.
