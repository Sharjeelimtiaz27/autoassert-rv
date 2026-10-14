# Candidate Metrics — Literature Notes (September 2026 pass)

> **Status after the advisor meeting (14 Oct 2026). Read this first.**
> - Kept as metrics: **SRTLC, AER, SAPC** (sections 1–3 below).
> - **TCFC (section 4) and the CVSS term (section 5) are no longer metrics.**
>   They are now comparison tables (assertion → CWE/CVSS, assertion →
>   TrustHub category). The 42 TCFC oracle runs are dropped.
> - **AQS is now `(SRTLC_contrib + AER + SAPC_contrib) / 3`.** The 5-term
>   formula in section 5 is out of date.
> - In AER, the injected free variable is now called a **test signal**
>   (`test_sig`), not "TROJ", because AER runs on the clean design.
> - New finding: arXiv 2201.01130 (the closest prior work to SAPC) is by
>   Eslami, **Ghasempouri** and Pagliarini (ISQED 2022), which is our own
>   group's earlier work. Present SAPC as an extension of it.
> - The current explanation is [metrics_explained.md](metrics_explained.md).
>   The deeper reviews are in [research/](research/).
>
> The notes below are kept as the record of the September literature pass.

Five candidate metrics for the AQS ranker. Lightweight literature pass only
(targeted web search, not the full deep-research skill) — treat sources
below as starting points to verify/expand, not a complete survey.

---

## 1. SRTLC — Security RTL Coverage

**Formula:**
```
SRTLC_M = (security-critical RTL lines inside union of assertion COIs) /
          (total security-critical RTL lines in module M) × 100%
```

**How it works:** Take every RTL line that implements a signal an NS31A
expert wrote a security assertion about. Read JasperGold's Cone-of-Influence
(COI) coverage output for the module's assertion set. SRTLC is the fraction
of those expert-flagged lines that fall inside some assertion's COI. It
answers: *of the RTL that experts consider security-relevant, how much is
actually being watched by an assertion?*

**Why the denominator matters:** It is NOT all RTL lines (that would just be
generic code coverage). It is only lines tied to NS31A-flagged signals, so
80% SRTLC means "80% of expert-identified security logic is monitored,"
not "80% of the module compiles under some assertion."

**Where the idea comes from:** COI-based coverage is standard JasperGold
output; the "restrict the denominator to expert/asset-flagged signals"
idea is closest in spirit to asset-based coverage metrics, e.g. asset
coverage in Trojan trust-metric work, and to **LAsset** (DATE 2026,
Farahmandi group) which identifies security *assets* via LLMs — SRTLC is
downstream of asset identification, using NS31A's existing expert
assertions as the asset list instead of an LLM. Also worth citing:
[Information Flow Coverage Metrics for Hardware Security Verification](https://arxiv.org/pdf/2304.08263)
— information-flow-based coverage criteria for hardware security, a
close conceptual cousin of restricting coverage to security-relevant paths.

---

## 2. AER — Assertion Evasion Resistance

**Formula:**
```
AER_A = 1  if JasperGold FPV proves no oracle value violates property P
            while assertion A stays silent
AER_A = 0  if JasperGold finds a counterexample (a concrete bypass trace)
```

**How it works (the "FPV oracle"):** For each security-critical signal, add
one free variable into the design — wired in alongside the real signal via
an XOR gate, so the free variable can flip the signal's value on any cycle.
JasperGold's FPV engine then tries every possible value of that free
variable, at every clock cycle, searching for one that causes a security
violation while the assertion under test produces no failure. If it can't
find one (Proven), the assertion is evasion-resistant. If it finds one
(CEX), that CEX is an exact recipe for a behavioral Trojan that slips past
that specific assertion.

Think of the free variable as standing in for "any possible hardware Trojan
that could exist at this signal" — you don't need to hand-design a Trojan;
JasperGold's solver explores the entire space of possible bad values for
you. This is a **behavioral** check: it asks "can a bad *value* get through
unnoticed," assuming the wire already exists.

**Where the idea comes from:** Directly extends the free-XOR-variable
generic hardware-Trojan model from **Chuah et al., "Formal Verification of
Security Properties on RISC-V Processors"** (RISC-V security-assertion
translation + Trojan testing, MBMV 2024; corrected Oct 2026) — their model tests
a module/PC-level property against an LLM-defined Trojan; AER reframes the
same free-variable mechanism as a **per-assertion** score, independent of
any specific Trojan.

**Differentiation vs. mutation testing (important, added after deeper pass):**
A close alternative technique is mutation testing — inject a finite set of
concrete RTL faults (stuck-at, operator inversions, boundary flips) and
measure the fraction the assertion set "kills." Two 2026 papers already do
this for RTL/assertion quality: [Closing the Loop on LLM-Generated RTL
Assertions with Quality-Aware Formal Verification](https://arxiv.org/pdf/2606.21451)
(90-100% kill rates, refines assertions on surviving mutants — functionally
close to this paper's own ARS loop, done generically) and
[GateTruth](https://arxiv.org/html/2608.12635) (semantic mutation operators,
kill-rate as rigor metric); mutation testing for Trojan detection specifically
goes back to [IEEE 7372619, 2015](https://ieeexplore.ieee.org/document/7372619/).
**Do not build a "mutation kill rate" 6th metric — it would land as
reinventing the 2026 papers, not a new contribution.** Instead, use this as
a differentiation line for AER: mutation testing samples a finite, discrete
set of faults and reports a kill-rate estimate; AER's free-variable FPV
oracle is **formally exhaustive** — it proves the absence of a bypass across
the entire continuous value space, not just the sampled mutants. That is a
strictly stronger completeness guarantee and a clean citable contrast.

---

## 3. SAPC — Security Assertion Path Coverage

**Formula:**
```
SAPC_M = (structural paths monitored by some assertion's COI) /
         (total structural paths found by SPV) × 100%
```

**How it works (the "SPV oracle"):** JasperGold's Security Path
Verification (SPV) engine does a **structural**, not behavioral, analysis:
starting from a security-critical signal, it traces every logical/electrical
path back to a primary input, regardless of what values those inputs
actually take. This produces a complete list of "any input that could ever
physically reach this signal." SAPC then checks how many of those paths
fall inside some assertion's cone of influence. A path with no assertion
watching it is a **structural blind spot** — a physically real route into
the security-critical signal that no property is even wired to see, whether
or not any bad value has ever been tried on it.

**Why AER alone is not enough:** AER=1 only proves no bad *value* got past
the assertion *among the inputs the FPV oracle was wired to*. It says
nothing about an input path that was never connected to the oracle model in
the first place. SAPC catches exactly that gap. AER is "no known escape
route exists"; SAPC is "no unmonitored road exists at all."

**⚠️ Novelty check — needs repositioning:**
[Reusing Verification Assertions as Security Checkers for Hardware Trojan
Detection](https://arxiv.org/abs/2201.01130) (IEEE conf. 2022, evaluated on
OpenTitan) already uses JasperGold SPV to define **"Security Coverage" =
reachable nodes / total nodes**, essentially the same path-reachability
idea, to judge whether an assertion can detect a Trojan at a given payload
location. **The current claim in CLAUDE.md ("first SPV-based assertion
quality metric in the literature") is not accurate as stated** and should
be revised before submission. A defensible differentiation: that paper
scores *whether a Trojan payload location is reachable to an assertion*
(Trojan-set-dependent framing); SAPC scores *structural coverage of
expert-defined security signals independent of any specific Trojan set*,
folded into a 5-term composite (AQS) validated against Trojan detection
counts across 9 RISC-V security modules. Discuss with advisor — this is
the single most important thing to get right before ETS submission, since
overclaiming novelty against a paper the reviewers likely know is a fast
way to get rejected.

---

## 4. TCFC — Trojan Category Formal Coverage (DROPPED: now a comparison table)

**Formula:**
```
TCFC_M = (TrustHub categories with a Proven category-oracle) / 6 × 100%
```

**How it works:** Build one FPV oracle per TrustHub attack category (Covert,
DoS, Integrity, Leak, Privilege, Availability) — each oracle is a
*generalized, category-level* Trojan model (not one of the 39 specific
benchmark Trojans), covering "any attack that fits the pattern of this
category." Run FPV against each. TCFC counts how many of the 6 categories
the module's assertion set formally defeats. This answers a different
question from SAPC/AER: not "is this one path/value monitored" but "does
the assertion set have blind categories entirely" — e.g. an assertion set
that's great against Leak and Integrity attacks but has zero coverage
against Covert-channel-style attacks.

**Where the idea comes from:** TrustHub's own attack taxonomy (categories
used are the standard TrustHub classes). The "one universal formal oracle
per category, run with FPV" framing appears to be your own construction —
this lightweight search did not surface a prior paper doing category-level
*formal* (not simulation-based) coverage against TrustHub's taxonomy
specifically. Worth a deeper check before claiming novelty, but this looks
like the most defensibly original of the five metrics from what surfaced.

---

## 5. AQS — Assertion Quality Score (OUT OF DATE: now the mean of SRTLC, AER, SAPC)

**Formula:**
```
AQS = 0.20×CVSS_norm + 0.20×AER + 0.20×SRTLC_contrib +
      0.20×SAPC_contrib + 0.20×TCFC_contrib
CVSS_norm = CVSS score of the assertion's mapped CWE / 10
```

**How it works:** Each assertion is mapped to a CWE weakness category (see
CLAUDE.md's per-module CWE table), and that CWE's CVSS severity score is
normalized to [0,1]. This is combined equally with the four formal metrics
above into one ranked score per assertion. Equal weighting (0.20 each) is
the default — worth a sensitivity-analysis paragraph in the paper (how much
does the ranking change if weights shift?).

**Where the idea comes from:** Using CVSS/CWE to weight software vulnerability
findings is standard practice (CWSS — Common Weakness Scoring System — does
this exact combination for software). Applying CVSS/CWE severity weighting
on top of formal hardware-verification coverage metrics, combined into one
composite score and then *validated empirically* by correlating the score
with real Trojan detection counts (Pearson correlation, §7c in the paper
plan) is the part that looks least covered by what surfaced in this pass —
that validation step is your strongest novelty claim of the five metrics,
independent of how SAPC's novelty framing gets resolved.

**Possible future refinement (not urgent):** [NIST CSWP 45, "Metrics and
Methodology for Hardware Security Constructs"](https://csrc.nist.gov/pubs/cswp/45/metrics-and-methodology-for-hardware-security-cons/final)
(June 2025) defines hardware-native *threat* (# hardware weaknesses one
CAPEC attack pattern can exploit) and *sensitivity* (# attack patterns that
can exploit one CWE weakness) metrics, derived purely from CWE/CAPEC
mapping — a hardware-specific alternative to borrowing CVSS from software.
Worth a citation now to justify the CVSS_norm term's grounding; swapping
CVSS_norm for a CSWP-45-style threat×sensitivity term is a reasonable
future refinement, not needed for this submission.

---

## Summary — what to tell the advisor

- **AER** (FPV oracle): solid, direct, well-scoped extension of Chuah's
  free-variable Trojan model to a per-assertion score. Low risk. Now also
  has a clean differentiation line vs. mutation testing (formally exhaustive
  vs. finite-sample kill rate) — strengthens rather than weakens it.
- **SAPC** (SPV oracle): the underlying mechanism (SPV path coverage) has
  prior art (arxiv 2201.01130) — reframe the novelty claim before
  submission, don't drop the metric, it's still useful and well-motivated
  alongside AER.
- **SRTLC**: reasonable, closest analog is asset-coverage/LAsset-style
  thinking — fine to keep, just don't call it entirely new.
- **TCFC**: looks like the most original piece structurally — the
  category-level universal FPV oracle idea.
- **AQS + Pearson-correlation validation against real Trojan detection**:
  likely the paper's actual strongest and most defensible contribution —
  formally-derived score empirically shown to predict real-world detection
  capability. This is worth leading with in the abstract/intro rather than
  "we invented 5 new metrics."

**On a 6th metric (checked and rejected):** mutation testing was the
obvious next candidate (inject RTL faults, measure assertion kill rate),
but two 2026 papers ([2606.21451](https://arxiv.org/pdf/2606.21451),
[GateTruth](https://arxiv.org/html/2608.12635)) already do this for RTL
assertion quality, and Trojan-specific mutation testing dates to 2015. A
standalone "mutation kill rate" metric here would read as reinventing that
work, not extending it. Recommendation: stay at 5 metrics; spend the extra
effort making AER's and SAPC's novelty claims precise instead of adding a
weak 6th one. If a 6th metric is still wanted, the CSWP-45 threat/sensitivity
angle (above, under AQS) is a more defensible direction than mutation
testing — it hasn't been combined with formal per-assertion coverage before,
as far as this pass found.
