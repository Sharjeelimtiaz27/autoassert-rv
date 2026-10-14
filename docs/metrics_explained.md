# The Assertion Quality Ranker — Explained

*Version 2, updated after the advisor meeting of 14 October 2026.*

**What changed in this version:** the ranker now has three metrics: SRTLC,
AER and SAPC. The former metrics 4 (Trojan category coverage) and 5 (CVSS
severity) are now **comparison tables**, not metrics. In AER, the injected
variable is called a **test signal**, because AER runs on the clean design
and no Trojan is involved.

**Purpose of this document:** explain the ranker from scratch, so it can be
read and discussed without prior knowledge of JasperGold FPV or SPV. Each
metric has its full name, a one-line intuition, numbered steps, the formula,
a small worked example and an honest note on where the idea comes from.

**The one-sentence pitch:** we have 68 security assertions for a RISC-V
processor that are already proven correct. Can we give each one a formal
quality score, before any hardware Trojan exists, and does that score
predict which assertions catch real Trojans later? That score is the
**Assertion Quality Score (AQS)**, and the three metrics below are its parts.

---

## Where the assertions come from

The input is **68 security assertions for the Ibex RISC-V core**, covering
all 9 security modules (PMP, CSRs, debug, exceptions and traps, control
flow, mode transition, memory access, instruction execution, register
update). They come from our earlier paper, *AutoTrans: AI-Assisted
Automatic Translation of Security Assertions for RISC-V Processors*
(BEC 2026). There, properties written for the NS31A processor (in SVA and in
English) were translated to Ibex and proven with JasperGold: every assertion
holds on the clean design and is non-vacuous (its trigger condition can
actually occur).

This paper does no translation. It takes those 68 assertions and asks how
good they are.

---

## Background: assertions and bind files

A **security assertion** is a short formal statement in SystemVerilog
Assertions (SVA) saying "this bad thing must never happen". For example:
"the machine status register must never be written while the processor is
in user mode". Assertions are placed in a separate **bind file** that
attaches them to the design without changing the design's source code.

**Cadence JasperGold** then proves, for every possible input sequence,
whether the assertion can be violated. There are two outcomes:

- **Proven:** the bad thing can never happen.
- **CEX (counterexample):** the tool found a concrete input sequence that
  makes it happen, and shows that sequence.

What a CEX means depends on what was being checked:

| Where it is run | What a CEX means |
|---|---|
| Clean design (original Ibex) | The assertion is wrong. This was already ruled out in the BEC paper. |
| Clean design + test signal (AER) | The assertion can be evaded: a gap in the assertion. |
| Trojan-infected design | The assertion caught the Trojan: a good result. |

All three metrics are computed on the **clean design**. Trojans are used
only afterwards, to check whether the score was right.

---

## The two JasperGold engines: FPV and SPV

The metrics rely on two different JasperGold analyses that answer different
questions.

### Formal Property Verification (FPV) checks behavior

FPV explores every value every input can take, on every clock cycle, and
proves whether any combination breaks a property. It is a mathematical
proof over all behaviors, not simulation or random testing.

> FPV asks: *"Is there any sequence of input values that breaks this?"*

### Security Path Verification (SPV) checks structure

SPV traces the paths through the logic that connect the chip's inputs to a
chosen signal. It does not try values. It looks at how the design is
connected.

> SPV asks: *"Which inputs have any route through the logic to this signal?"*

### Why both are needed

Think of a bank vault. FPV asks whether any code typed on the keypad opens
the door. SPV asks whether there is an air duct or a side door anywhere in
the building that leads into the vault. A vault can pass the first check
and fail the second. **AER** is the keypad check (FPV) and **SAPC** is the
building-plan check (SPV).

---

## Metric 1 — Security RTL Coverage (SRTLC)

**One-line intuition:** of all the hardware logic that experts consider
security-relevant, how much is watched by at least one assertion?

**Steps:**
1. Take the NS31A expert security properties. Every signal they mention is
   called **security-critical**, because a domain expert wrote a security
   property about it. No keyword search or guessing is involved.
2. Find the Ibex signals that correspond to them. Signals that could not be
   mapped still count, so a missing mapping shows up as a coverage gap.
3. For each security-critical signal, list the RTL lines that drive it
   (`always` blocks, `assign` statements, conditions).
4. Read JasperGold's **cone-of-influence (COI)** report for the assertion
   set. The COI of an assertion is all the logic that can affect it, which
   is the logic it "watches".
5. SRTLC is the share of security-critical lines that lie inside at least
   one assertion's COI.

**Formula:**
```
SRTLC_M = security-critical RTL lines inside the union of assertion COIs
          ------------------------------------------------------------  × 100%
          all security-critical RTL lines in module M
```
For ranking a single assertion, **SRTLC_contrib** is the share of
security-critical lines that only this assertion covers.

**Worked example:** the CSR module has 40 lines that drive expert-flagged
signals. The assertions' cones of influence cover 30 of them.
SRTLC = 30/40 = **75%**. We also know exactly which 10 lines are watched by
no assertion at all.

**Why the denominator is not "all RTL lines":** if we divided by the whole
module (say 2,000 lines), the security gap would disappear into a
misleading general coverage figure. Counting only expert-flagged lines keeps
the number about security.

**Origin and novelty:** COI coverage is standard JasperGold output. Limiting
it to expert- or asset-defined signals is close in spirit to security asset
identification (LAsset, DATE 2026) and to information-flow coverage metrics
(arXiv 2304.08263). It is a sound, well-motivated metric. Present it as an
adaptation, not as a brand-new idea.

---

## Metric 2 — Assertion Evasion Resistance (AER)

**One-line intuition:** if a security-critical signal could be flipped at
any moment, in any pattern, could that cause a security violation without
the assertion noticing?

**Steps (the "test signal" setup):**
1. Choose one security-critical signal, `real_signal`.
2. Add one new input to the design: a **test signal**, `test_sig`. Nothing
   drives it, so JasperGold may set it to any value on any clock cycle.
3. Combine them: `modified_signal = real_signal XOR test_sig`. When
   `test_sig = 0`, nothing changes. When `test_sig = 1`, the signal's value
   is flipped on that cycle.
4. Feed `modified_signal` into the rest of the design in place of
   `real_signal`. The design itself is unchanged; the test signal sits in a
   small wrapper.
5. Ask JasperGold FPV: *is there any sequence of test-signal values that
   breaks the intended security property while this assertion stays
   silent?*
6. Read the result:
   - **Proven:** no such sequence exists, so the assertion cannot be evaded.
     **AER = 1.**
   - **CEX:** JasperGold returns the exact test-signal sequence that slips a
     violation past the assertion. **AER = 0**, and that trace shows
     precisely where the assertion is weak.

**Formula:**
```
AER_A = 1  if FPV proves no test-signal sequence breaks the security
           property while assertion A stays silent
AER_A = 0  if FPV returns a counterexample (an evasion trace)
```

**Worked example:** the assertion `csr_no_write_in_user_mode` should catch
any write to the machine status register from user mode. We attach a test
signal to the register's write-enable and ask FPV whether flipping it can
cause such a write while the assertion stays quiet. If the answer is
Proven, AER = 1. If FPV shows the test signal forcing write-enable high on
cycle 4 while the assertion output stays low, AER = 0, and we know exactly
which pattern evades it.

**Why this is stronger than testing a few cases:** the test signal can take
every value on every cycle, so a Proven result covers every possible
tampering pattern on that signal, not just the ones someone thought of.

**Origin and novelty:** the free-variable idea comes from the generic
tampering model of Chuah et al., *Formal Verification of Security
Properties on RISC-V Processors* (MBMV 2024) and their RISC-V Summit Europe 2025 work, which checks a whole
module. AER turns the same mechanism into a **score for each individual
assertion**.

A nearby technique is **mutation testing**: insert a fixed list of concrete
faults and measure what fraction the assertions catch (a "kill rate").
Two 2026 papers already do this for RTL assertions
([arXiv 2606.21451](https://arxiv.org/pdf/2606.21451) and
[GateTruth, arXiv 2608.12635](https://arxiv.org/html/2608.12635)). AER
differs because it is a **proof over all test-signal values**, not an
estimate from a sample of faults. That is the honest way to set AER apart
from that work.

---

## Metric 3 — Security Assertion Path Coverage (SAPC)

**One-line intuition:** even if no tampered value gets past an assertion,
is there a route into the security signal that no assertion watches at all?

**Steps:**
1. Take the same security-critical signals.
2. Run JasperGold SPV and ask for every structural path from a chip input to
   each signal. This is a wiring analysis; no values are tried.
3. For each path, check whether it lies inside some assertion's cone of
   influence (the same COI data used for SRTLC). If yes, the path is
   **monitored**. If no assertion's COI includes it, it is a **structural
   blind spot**: a real route into the security signal that no assertion
   watches.
4. SAPC is the share of paths that are monitored.

**Formula:**
```
SAPC_M = SPV paths into security signals inside some assertion's COI
         -----------------------------------------------------------  × 100%
         all SPV paths into security signals
```
For ranking a single assertion, **SAPC_contrib** is the share of paths
that this assertion's COI covers.

**Worked example:** SPV finds 8 inputs with a path into the machine status
register. Six of those paths are inside some assertion's COI and 2 are not.
SAPC = 6/8 = **75%**. This can happen even when every assertion has
AER = 1, because AER only tampers with the signal the test signal is
attached to. The 2 unwatched paths are invisible to AER. SAPC finds them
because it looks at the wiring, not at values.

**Why AER and SAPC belong together:** AER says *"no tampered value gets
past the assertions we checked"*. SAPC says *"no route into the signal is
left unwatched"*. They catch different kinds of blind spot, and a real
assertion set can be strong on one and weak on the other.

**Origin and novelty, stated honestly:** a 2022 paper,
[Reusing Verification Assertions as Security Checkers for Hardware Trojan
Detection (arXiv 2201.01130)](https://arxiv.org/abs/2201.01130), already
uses JasperGold SPV to define "Security Coverage = reachable nodes / total
nodes" for assertions. So we must **not** claim to be the first to use SPV
for assertion coverage. The difference is that their measure is defined for
a candidate Trojan location, while SAPC is defined for each expert-flagged
security signal with no Trojan in mind, and is then combined with SRTLC and
AER and checked against real Trojan detection.

---

## The Assertion Quality Score (AQS)

**One-line intuition:** combine the three metrics into one number per
assertion, so the 68 assertions can be ranked from strongest to weakest.

**Formula:**
```
AQS = (SRTLC_contrib + AER + SAPC_contrib) / 3        (between 0 and 1)
```
All three parts are fractions between 0 and 1 and get equal weight.

**Worked example:** an assertion with SRTLC_contrib = 0.6, AER = 1 and
SAPC_contrib = 0.5 gets AQS = (0.6 + 1 + 0.5) / 3 = **0.70**.

**How we test the score:** every assertion gets its AQS on the clean design.
Then the same assertions are run against the 39-Trojan benchmark, and we
count how many Trojans each one detects. We compute the **Pearson
correlation** between AQS and the detection count. A strong positive
correlation supports the central claim of the paper: *a formal score,
computed before any Trojan existed, predicts which assertions catch real
Trojans.*

---

## Comparison tables (not metrics)

Two tables sit beside the AQS ranking to describe each assertion. They do
not affect the score.

### Table A — assertion to weakness (CWE / CVSS)

Each assertion is mapped to the CWE weakness it guards against, with that
weakness's CVSS severity. Example: PMP assertions map to CWE-1220
(insufficient granularity of access control). This shows *what kind of
weakness* each assertion protects, and how serious that weakness is.

Cite **NIST CSWP 45**, *Metrics and Methodology for Hardware Security
Constructs* (June 2025), which defines hardware-specific *threat* and
*sensitivity* measures from CWE and attack-pattern (CAPEC) mappings, to
ground this table in hardware-security literature.

### Table B — assertion to Trojan category

Each assertion is mapped to the TrustHub Trojan categories it is meant to
address: Availability, Covert, Denial of Service, Integrity, Leak and
Privilege. After the Trojan evaluation, the table also shows which
categories each assertion *actually* detected.

---

## The full flow in five stages

| # | Stage | What happens | New JasperGold runs |
|---|---|---|---|
| 1 | Assertion Validation (AVS) | Done in the BEC 2026 paper: 68 assertions proven and non-vacuous | 0 |
| 2 | Assertion Quality Ranking (AQRS) | SRTLC, AER and SAPC on the clean design, then AQS and the two comparison tables | 20 |
| 3 | Trojan Evaluation (TES) | Run the assertions on 39 trojaned designs, then correlate AQS with detection | 39 |
| 4 | Assertion Refinement (ARS) | Add template assertions for the gaps found in stage 2 (Python, no LLM) | 0 |
| 5 | Refined Assertion Validation (RAVS) | Re-run on the same 39 Trojans and measure the improvement | 39 |

---

## Novelty summary for discussion

| Item | Status | What to say |
|---|---|---|
| SRTLC | Sound, not brand-new | "Coverage of expert-defined security logic, adapted from asset and information-flow coverage." |
| AER | Strong | "A per-assertion formal proof of evasion resistance; a proof over all test-signal values, not a mutation kill rate." |
| SAPC | Claim must be reworded | "Not the first SPV metric (arXiv 2201.01130). New: defined per security signal, independent of any Trojan, and combined with AER and SRTLC." |
| AQS + correlation | Strongest claim | "A formal score computed before any Trojan exists predicts real Trojan detection." Lead the abstract with this. |
| CWE and Trojan-category tables | Descriptive | Context for the reader; not claimed as novel. |

The red-team, blue-team and literature reports in `docs/research/` check
these verdicts in more depth.
