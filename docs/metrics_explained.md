# AutoAssert-RV: The Assertion Quality Ranker — Explained

**Purpose of this document:** explain, from scratch, the five metrics that
make up our security assertion quality ranker (AQS), for advisor discussion.
No prior familiarity with JasperGold FPV/SPV is assumed. Each metric is
given in full, with its abbreviation, a plain-language intuition, the exact
mechanical steps a reader could reproduce, the formula, and a small worked
example with made-up numbers so the arithmetic is concrete.

**The one-sentence pitch:** Given a set of security assertions (SVA
properties) written for a RISC-V processor, can we compute, before ever
running a single hardware Trojan, a formal score that tells us which
assertions are strong and which are weak — and does that score actually
predict which assertions catch real Trojans when we test them? That formal
score is **AQS**. The five metrics below are its five ingredients.

---

## Background you need first: what is an "assertion" and a "bind file"?

A **security assertion** is a small formal statement written in SystemVerilog
Assertions (SVA) that says "this bad thing must never happen" — e.g. "the
machine-mode status register must never change value while the processor is
in user mode." Each assertion is placed in a separate **bind file** (never
inside the processor's own source code) that attaches it to the RTL design
without modifying the design itself.

A formal verification tool (we use **Cadence JasperGold**) then
mathematically proves, for every possible input sequence the hardware could
ever receive, whether the assertion can be violated. Two outcomes:
- **Proven** — the tool proved the bad thing can never happen (assertion holds).
- **CEX (counterexample)** — the tool found a concrete sequence of inputs
  that makes the bad thing happen (assertion is violated), and shows you
  exactly what that sequence is.

Whether "CEX" is good or bad news **depends on what RTL you ran it on** —
this trips people up, so it's worth stating plainly up front:
- Run on the **clean, un-modified** processor → CEX means the assertion is
  *wrong* (a translation/writing bug) — bad news, fix the assertion.
- Run on a **Trojan-infected** copy of the processor → CEX means the
  assertion *caught the Trojan* — good news, the assertion works.

Everything below happens on clean RTL only, except the final Trojan
evaluation stage, which is what validates the whole ranker.

---

## The two formal engines behind everything: FPV and SPV

JasperGold offers two different formal analysis modes, and it is critical
not to blur them together, because they answer fundamentally different
questions.

### JasperGold Formal Property Verification (FPV) — a BEHAVIORAL check

**What it does:** Given a property (an assertion) and a design, FPV explores
*every possible value* every input signal could take, on every clock cycle,
forever, and mathematically proves whether any combination of those values
could violate the property. It is exhaustive — not simulation, not random
testing, an actual mathematical proof over the full state space.

**The question FPV answers:** *"Is there ANY sequence of input VALUES that
breaks this property?"*

### JasperGold Security Path Verification (SPV) — a STRUCTURAL check

**What it does:** Given a target signal (e.g. a security-critical register),
SPV traces every logical/electrical wire path that connects a primary input
of the chip to that target signal — completely independent of what values
those inputs take. It's asking a wiring-diagram question, not a
"what-if-this-input-were-5" question.

**The question SPV answers:** *"Which inputs have ANY physical route
(through gates, muxes, wires) to this signal at all?"*

### Why the difference matters — a one-paragraph analogy

Think of a bank vault. FPV asks: "if someone tries every possible
combination on the keypad, can they ever open the door?" SPV asks: "is
there a maintenance duct, an air vent, or a second door anywhere in the
building that physically connects to the vault, whether or not anyone has
tried it yet?" A vault can pass the first test (no combination opens it)
while still failing the second (there's an unguarded duct straight into the
vault). Our two novel metrics, **AER** (built on FPV) and **SAPC** (built on
SPV), are exactly this pair of checks, applied to security assertions.

---

## Metric 1 — Security RTL Coverage (SRTLC)

**Full name:** Security RTL Coverage
**One-line intuition:** Of all the hardware logic that experts consider
security-relevant, how much of it is actually being watched by at least one
assertion?

**Step-by-step mechanics:**
1. Start from NS31A (the reference/source processor) expert-written
   security assertions. Every signal one of those assertions mentions is
   declared "security-critical" — not by keyword search, not by guessing,
   but because a domain expert wrote a formal security property about it.
2. Map that list of security-critical signals onto their Ibex RTL
   equivalents (some map cleanly, some don't — the ones that don't still
   count in the denominator, see step 5).
3. For each security-critical signal, find every RTL line (an `always`
   block, an `assign` statement, a conditional) that implements it.
4. Run JasperGold's Cone-of-Influence (COI) coverage report for the
   module's actual assertion set — this tells you, for the assertions we
   have, exactly which RTL lines they "see" (are logically connected to).
5. SRTLC is the fraction of security-critical RTL lines that fall inside
   the union of all assertion COIs.

**Formula:**
```
SRTLC_M = (security-critical RTL lines inside union of assertion COIs)
          -----------------------------------------------------------  × 100%
          (total security-critical RTL lines in module M)
```

**Worked example:** Suppose the CSR module has 40 RTL lines that implement
NS31A-flagged security signals. The module's assertion set's combined COI
touches 30 of those 40 lines. SRTLC = 30/40 × 100% = **75%**. This tells us
25% of expert-identified security logic has *zero* assertion watching it at
all — a concrete, actionable gap (we'd know exactly which 10 lines).

**Why the denominator is NOT "all RTL lines":** if we divided by the whole
module (e.g. 2,000 lines), 75% coverage of security logic could disappear
into a misleadingly high "98% RTL coverage" number that says nothing about
security. Restricting the denominator to expert-flagged lines keeps the
metric meaningful.

**Source of the idea / how novel:** Cone-of-influence coverage itself is
standard JasperGold output. Restricting the denominator to expert- or
asset-identified signals (rather than all RTL) is closest in spirit to
**LAsset** (DATE 2026, Farahmandi group, University of Florida) — they use
an LLM to identify security "assets"; we use NS31A's existing expert
assertions as the asset list instead. Not a brand-new idea, but a
reasonable, well-motivated one — present it as such, not as groundbreaking.

---

## Metric 2 — Assertion Evasion Resistance (AER)

**Full name:** Assertion Evasion Resistance
**One-line intuition:** If a hardware Trojan could flip any bit of this
security-critical signal, at any time, in any pattern — could it get away
with it without the assertion noticing?

**Step-by-step mechanics ("the FPV oracle"):**
1. Pick one security-critical signal, call it `real_signal`.
2. Add one brand-new, completely free (unconstrained) input to the design —
   call it `TROJ`. This variable is not driven by any real logic; JasperGold
   is free to set it to anything, at any cycle.
3. Wire it in: `modified_signal = real_signal XOR TROJ`. When `TROJ = 0`,
   nothing changes. When `TROJ = 1` at some cycle, the signal's value flips
   at that cycle — this is a stand-in for "a Trojan just tampered with this
   signal."
4. Feed `modified_signal` (not the original) into the rest of the design,
   in place of `real_signal`.
5. Run JasperGold FPV and ask: *does there exist ANY sequence of values for
   `TROJ`, across ANY number of cycles, that causes the underlying security
   property to be violated, while the security assertion we're testing
   produces no failure (stays silent)?*
6. Two outcomes:
   - **Proven** → no such `TROJ` sequence exists → the assertion is
     evasion-resistant → **AER = 1**.
   - **CEX** → JasperGold hands you the *exact* sequence of `TROJ` values
     that sneaks a violation past the assertion → **AER = 0**, and you now
     have a concrete example of the exact behavioral gap.

**Formula:**
```
AER_A = 1   if JasperGold FPV proves no TROJ assignment violates
            the security property while assertion A stays silent
AER_A = 0   if JasperGold finds a counterexample (exact bypass shown)
```

**Worked example:** Assertion `csr_no_write_in_user_mode` should catch any
unauthorized write to the machine-status register. We wire a free `TROJ`
variable into the register's write-enable signal and ask FPV: can `TROJ`
ever cause an unauthorized write that this assertion fails to flag? If
JasperGold returns Proven, AER = 1 for this assertion — it's a "yes" in a
detection-matrix column. If JasperGold returns a counterexample showing
`TROJ` forcing the write-enable high on cycle 4 while the assertion output
stays low, AER = 0, and you know exactly which value pattern defeats it.

**Why this matters beyond a single Trojan test:** this is not "did our 39
specific Trojans get caught" — it is "is there ANY possible way, out of the
literally infinite space of possible tampering patterns, to defeat this
specific assertion." It is exhaustive by construction, not a sample.

**Source of the idea:** directly extends the generic free-variable hardware
Trojan model from **Chuah et al., "Formal Verification of Security
Properties on RISC-V Processors"** — their model checks a property against
one free-variable-modeled Trojan at module/PC level. AER's contribution is
reframing the same mechanism as a **per-assertion score** you compute for
every single assertion, independently, rather than a pass/fail for the
whole module.

**Differentiation vs. a nearby alternative technique (mutation testing):**
a natural alternative is "mutation testing" — inject a fixed, finite list
of concrete faults (stuck-at-0, stuck-at-1, flipped comparison operators,
etc.) and measure what fraction the assertions catch (a "kill rate"). Two
2026 papers already do this for RTL assertion quality generally
([arXiv 2606.21451](https://arxiv.org/pdf/2606.21451),
[GateTruth](https://arxiv.org/html/2608.12635)). AER is a stronger
guarantee than any of those: mutation testing samples a finite set of
faults and reports what fraction survive; AER's free-variable FPV oracle
proves, mathematically, that **no** value in the entire continuous space
causes a bypass — a proof of absence, not an estimate from samples. This is
a clean, honest way to differentiate AER from that line of work rather than
pretending it doesn't exist.

---

## Metric 3 — Security Assertion Path Coverage (SAPC)

**Full name:** Security Assertion Path Coverage
**One-line intuition:** Even if an assertion perfectly catches every bad
*value* we've thought to test (AER = 1), is there some *physical wire* into
the security-critical signal that no assertion is even watching?

**Step-by-step mechanics ("the SPV oracle"):**
1. Pick the same security-critical signal used for AER.
2. Run JasperGold SPV, telling it: "trace every possible structural
   (logical/electrical) path from any primary input of the chip to this
   signal." SPV returns a complete list of such paths — this is a wiring
   analysis, it does not involve trying any input values.
3. For each path SPV found, check: does any assertion's cone-of-influence
   (from the same COI data used for SRTLC) include this path? If yes, the
   path is "monitored." If no assertion's COI touches it, the path is a
   **structural blind spot** — a real, physically existing route into the
   security signal that literally no property is watching.
4. SAPC is the fraction of SPV-found paths that are monitored.

**Formula:**
```
SAPC_M = (structural paths monitored by some assertion's COI)
         ----------------------------------------------------  × 100%
         (total structural paths found by SPV)
```

**Worked example:** SPV finds that 8 distinct primary inputs have a
structural path into the machine-status register. Checking assertion COIs,
6 of those 8 paths are inside some assertion's cone of influence; 2 are not
watched by anything. SAPC = 6/8 × 100% = **75%**. Crucially, this can be
true *even if AER = 1 for every existing assertion* — because AER only
tested the specific inputs its oracle model was wired to. If the oracle was
never wired into those 2 unmonitored paths in the first place, AER would
never have found the gap. SAPC finds it because it doesn't depend on the
oracle wiring at all — it's a pure wiring-diagram trace.

**Why you need both AER and SAPC — this is the paper's central insight:**
AER = "no known escape route exists, among the routes we tested." SAPC =
"no unmonitored road exists, period — including roads we never thought to
test." They are answering different questions (behavioral vs. structural),
and a real assertion set can be strong on one and weak on the other.

**Source of the idea / honest novelty status:** JasperGold SPV path-tracing
is a standard, existing Cadence product feature (in commercial use since
2013). The specific idea of using SPV to score whether assertions cover
structural paths is **not new** — a close prior paper, [Reusing
Verification Assertions as Security Checkers for Hardware Trojan Detection
(arXiv 2201.01130, evaluated on OpenTitan)](https://arxiv.org/abs/2201.01130),
already defines "Security Coverage = reachable nodes / total nodes" using
the same JasperGold SPV mechanism, to judge whether an assertion can detect
a Trojan at a specific candidate payload location.

**What actually is different about our version:** their metric is defined
*per candidate Trojan payload location* — it presupposes you already have a
specific Trojan site in mind. SAPC is defined *per expert-flagged security
signal*, entirely independent of any specific Trojan or Trojan location, and
it is one ingredient of a 5-term composite score (AQS) that we then validate
empirically by correlating it with real detection counts across 9 modules
and 39 Trojans. **The honest claim is: the composite ranker + its empirical
validation is the novel part, not "we are first to use SPV for coverage."**
This needs to be stated this way in the paper — claiming "first SPV-based
metric" would be an easy, damaging thing for a reviewer who knows this 2022
paper to catch.

---

## Metric 4 — Trojan Category Formal Coverage (TCFC)

**Full name:** Trojan Category Formal Coverage
**One-line intuition:** Does this module's assertion set have an entire
*category* of attack it can't catch at all — not just one specific Trojan,
but a whole class of them?

**Background — the 6 categories:** TrustHub (the standard hardware Trojan
benchmark/taxonomy source) classifies attacks into 6 categories: Covert
(hidden channel), Denial-of-Service, Integrity (data tampering), Leak
(information disclosure), Privilege (unauthorized escalation), and
Availability. Real Trojans are usually specific instances of one of these
general categories.

**Step-by-step mechanics:**
1. For each of the 6 TrustHub categories, build one **universal formal
   oracle** — a JasperGold FPV model that represents *any possible attack
   fitting that category's general pattern*, not one of the 39 specific
   benchmark Trojans. (Example: the "Leak" category oracle is a generic
   model of "some internal secret value becomes observable on an output it
   shouldn't reach," not any one specific leak Trojan's implementation.)
2. Run FPV: does the module's existing assertion set catch every possible
   attack fitting this category's oracle?
3. Repeat for all 6 categories, independently.
4. Count how many of the 6 categories come back Proven (fully covered).

**Formula:**
```
TCFC_M = (TrustHub categories where the category oracle is Proven) / 6 × 100%
```

**Worked example:** Testing the WB (write-back) module's assertions against
all 6 category oracles: Integrity, Leak, and Privilege come back Proven
(assertions catch any attack of these types). Covert, DoS, and Availability
come back with counterexamples (some attack pattern of these types slips
through). TCFC = 3/6 × 100% = **50%**. This tells the module's assertion
author, in plain terms: "your assertions have a structural blind category —
you have zero formal coverage against Covert-channel-style attacks, no
matter which specific Covert Trojan someone eventually builds."

**Why this differs from AER and SAPC:** AER and SAPC are both about a
*single signal*, checked one at a time. TCFC operates at the *category
level, across the whole module* — it's the only metric of the five that
asks "which entire class of threat is unaddressed," rather than "which
specific signal or path is unwatched."

**Source of the idea / novelty:** the 6-category taxonomy comes from
TrustHub, an established public resource. Building one *universal, formal
(FPV-based)* oracle per category — as opposed to testing against specific
Trojan instances — did not turn up a close prior match in this pass of
searching. Of the five metrics, this currently looks like the most
originally-constructed one, though it deserves a deeper, dedicated
literature check before that claim is stated confidently in the paper.

---

## Metric 5 — Assertion Quality Score (AQS) — the composite ranker

**Full name:** Assertion Quality Score
**One-line intuition:** Combine everything above, plus how *dangerous* the
underlying weakness would be if exploited, into one single number per
assertion, so assertions can be ranked from strongest to weakest.

**Step-by-step mechanics:**
1. Take the four formal metrics above, each already expressed as a fraction
   between 0 and 1 for a given assertion: AER (0 or 1), SRTLC_contrib (this
   assertion's unique share of the module's SRTLC coverage), SAPC_contrib
   (this assertion's share of monitored structural paths), TCFC_contrib
   (fraction of the 6 categories this assertion contributes to).
2. Separately, map the assertion's module to a CWE (Common Weakness
   Enumeration) category — a standard taxonomy for classifying the *type*
   of security weakness (e.g. "CWE-1220: Insufficient Granularity of Access
   Control" for PMP). Each CWE has a published CVSS (Common Vulnerability
   Scoring System) severity score, 0–10. Normalize it: `CVSS_norm = CVSS/10`.
3. Combine all five terms with equal weights (0.20 each, chosen as the
   simplest defensible default — summing to 1.0):

**Formula:**
```
AQS = 0.20 × CVSS_norm
    + 0.20 × AER
    + 0.20 × SRTLC_contrib
    + 0.20 × SAPC_contrib
    + 0.20 × TCFC_contrib
```

**Worked example:** An assertion in the PMP module (CWE-1220, CVSS ≈ 8.6,
so CVSS_norm = 0.86) that is evasion-resistant (AER = 1), contributes 0.6 of
its module's unique SRTLC coverage, 0.5 of its SAPC-monitored paths, and 0.4
of the 6 TCFC categories:
```
AQS = 0.20(0.86) + 0.20(1.0) + 0.20(0.6) + 0.20(0.5) + 0.20(0.4)
    = 0.172 + 0.200 + 0.120 + 0.100 + 0.080
    = 0.672
```
This assertion would rank at 0.672 out of a possible 1.0, and can be
compared directly against every other assertion in every other module —
producing a single ranked table across the whole processor.

**Then — the step that actually proves the ranker means something:** once
every assertion has an AQS score (computed entirely on clean RTL, with no
Trojans involved), the same assertions are run against the 39-Trojan
benchmark (Trojan Evaluation Stage). For each assertion, count how many of
the 39 Trojans it actually detects. Then compute the **Pearson correlation
coefficient** between AQS score and detection count, across all assertions.
A strong positive correlation is the scientific claim of the whole paper:
*"a purely formal score, computed before any Trojan ever existed, predicts
which assertions will catch real Trojans."* This is the part of the paper
that should be led with — it's a stronger and more defensible claim than
any single one of the five ingredient metrics on its own.

**Source of the severity term / citations to use:**
- Using CWE-mapped CVSS severity to weight findings is standard practice in
  software security (**CWSS**, the Common Weakness Scoring System, does the
  same combination for software vulnerabilities).
- For a hardware-native alternative/complement, cite **NIST CSWP 45,
  "Metrics and Methodology for Hardware Security Constructs"** (June 2025):
  it defines a *threat* metric (how many hardware weaknesses one CAPEC
  attack pattern can exploit) and a *sensitivity* metric (how many attack
  patterns can exploit one CWE weakness), built entirely from CWE/CAPEC
  mapping rather than borrowed CVSS. This is a good citation to ground the
  severity-weighting idea in hardware-specific literature. Swapping
  CVSS_norm for a CSWP-45-style threat×sensitivity term is a reasonable
  *future* refinement — not necessary for this submission, but worth a
  sentence in the limitations/future-work section.

---

## Novelty verdict — one table, for the advisor conversation

| Metric | Status | What to say | Key citation to handle carefully |
|---|---|---|---|
| SRTLC | Reasonable, not brand-new | "Adapts asset-coverage-style thinking to expert-defined signals" | LAsset (DATE 2026) |
| AER | Solid, well-differentiated | "Formally exhaustive per-assertion extension of Chuah's free-variable model; strictly stronger than mutation-testing kill rates" | Chuah (RISC-V security props); 2606.21451, GateTruth (contrast) |
| SAPC | Needs reframed novelty claim | "Do not say 'first SPV-based metric' — say the composite+validation is novel, not the SPV mechanism itself" | **arXiv 2201.01130 (OpenTitan) — must cite and differentiate** |
| TCFC | Most originally-constructed, unverified | "Looks original from this pass; deserves a dedicated deeper check before claiming novelty confidently" | none found yet |
| AQS + Pearson validation | Strongest overall claim | "Lead the abstract with this: a formal score, computed with zero Trojans involved, empirically predicts real detection capability" | CWSS (software analog); NIST CSWP 45 (hardware-native option) |

---

## What this document is NOT

This was a lightweight-to-moderate literature pass (targeted web searches,
not the installed 13-agent deep-research tool, and not a systematic
review). Treat every "no prior work found" statement above as "nothing
turned up in this pass," not "confirmed absent from the literature." Before
final submission, at minimum SAPC and TCFC deserve one more focused check
each.
