# F4 row scope: what D9's matrix is indexed by

**Blocks:** D9, which cannot open against a row set
known to be incomplete.
**Probe:** [probes/derived-rows.sh](probes/derived-rows.sh) →
[.out](probes/derived-rows.out). Unauthenticated, no account, no key.

F4 fixed ten primitive rows: trader identity, KYC credential, order size, order
price, execution price, quantity, asset, compliance validity, settlement
validity, governance actions. Four independent sightings then found that the
damaging disclosure at a real venue is a *derived* fact over those rows rather
than a row itself: MK-009 credential history, MK-011 counterparty relationships,
MK-013 the match predicate, MK-020 `liquidationPx`. AB-033 called the fix cheap
and left the scope question to D9. This file settles it.

**The method note.** The four sightings are observations about other people's
venues. Before deciding scope from them, the same question was asked of *our*
asset class on mainnet today. That probe found five reconstructible derived
facts, three of which the four sightings did not predict, and one of which is
not a row at any granularity. The scope decision below is driven by the
measurement, not by the taxonomy.

---

## What the probe found

`[MEASURED]` treasury `0.0.10116630`, six tokens, 190 token
transactions, no credentials used.

| # | Derived fact | Function of | Scope |
|---|---|---|---|
| 1 | counterparty graph | identity, quantity | trader |
| 2 | activity fingerprint | identity, quantity, time | trader |
| 3 | fund flow, per share class | quantity, asset | **issuer** |
| 4 | submission routing | nothing in F4 | **transport** |
| 5 | cadence | time | trader |

Two incidental findings worth carrying. The shared treasury serves **five**
competing managers, not three: abrdn, BlackRock, State Street, Fidelity and
LGIM. MK-011 undercounted. And one of the six tokens is `US912797UJ40`, a
**tokenised US Treasury bill**, 216,014,405 units issued in a single transfer.
That is repo collateral, on Hedera, on mainnet, and it is a better pitch anchor
than the money market funds.

### The routing finding, which was not anticipated by anything

`[MEASURED]` The receiving consensus node is a **public field on every
transaction**. All 190 treasury transactions were submitted to node `0.0.29`.
That node is operated by Aberdeen Investments, which is one of the five managers
whose funds sit on that treasury.

Control: across 100 random recent mainnet transactions the traffic spreads over
28 distinct nodes with the busiest at 7.0 percent. So 100 percent on one node is
a deliberate pin, not an SDK default.

Stated without allegation, because the structural point does not need one:
**every transaction on this treasury, across five competing managers, is handed
first to a node operated by one of them**, and the fact that it was is permanently
public. This is C-03 and C-05 from [μ1.4](../mesh/consensus-visibility.md)
arriving together as a live instance. It also de-risks D-15: node pinning is not
an exotic idea we invented, it is existing production practice on this exact
asset. What is missing in production is that the pin is not disclosed as a
disclosure.

---

## The scope rule

Choosing rows by "how bad is this leak" produces an unbounded list and an
argument in a meeting. One rule, applied mechanically:

> **A derived row is in scope for D9 if and only if our own design determines
> whether it leaks.**

Everything else is out of the matrix and is instead **stated as a known
residual**, with the reason. This ties scope to a decision already taken rather
than to taste: the zero-fork claim (D-05) puts the ATS and HTS layers outside our
control by choice, so leaks originating there are residuals we disclose, not
cells we fill. A matrix that pretends otherwise would be claiming authority the
design does not have.

---

## The decision: seven derived rows, in scope

D9's matrix is **seventeen rows**: F4's ten primitives, plus these.

| Row | Value is | Why in scope | Mechanism available |
|---|---|---|---|
| **R11 credential history** | fn(KYC credential, time) | our proof plus nullifier replaces the register; if we get this wrong it is our fault | per-epoch nullifier, F5b |
| **R12 counterparty relationship** | fn(identity, quantity) | **the hardest row.** The §1.3 split settles in the clear against ATS, so settlement discloses the pair. Ours by construction | open. This row is the one that may force D-08's roadmap shape earlier |
| **R13 match predicate** | fn(order size, order price, both sides) | the venue computes the match; nothing else can leak it | one Groth16 public output. AB-033 |
| **R14 position risk** | fn(quantity, price, haircut, margin) | MK-020's `liquidationPx`. In a repo the analogue is the margin call trigger | do not publish the inputs. A design constraint, not a circuit |
| **R15 activity fingerprint** | fn(identity, quantity, time) | ticket-size distribution and breadth across instruments; the venue decides what it writes | fixed-size commitments, μ1.2 |
| **R16 cadence** | fn(time) | consensus timestamps cannot be hidden, but *when we write* is ours | ~~epoch batching, μ1.3~~ **open.** μ1.3 closed 2026-09-02, AB-058: batching cannot coarsen a cadence the venue produces at ~1.6 orders a day, so the cell is `(exact, {pub}, imm)`. Cover traffic is the mechanism, `BUILD-PLAN.md` §7.6 |
| **R17 account provenance** | fn(account creation ordinal, network creation rate) | `[ADDED after AB-035]` Hedera account numbers are monotonic, so a registration record dates its own account and an address created shortly before it registers is visibly purpose-made. We create those accounts, so it is ours | open. Per-epoch nullifiers do not touch it, `[SOURCE]` AB-023 |

R17 is a second sighting of one defect, not a new one: AB-023 found cohort
membership visible at account creation, AB-035 found the same ordinal dating an
individual registration. Both compose by join, which is why it is a row rather
than a footnote.

**R16 joined R12 as an open cell.** The mechanism column said
"epoch batching" and the mechanism does not exist: nothing in the design batches
order submission, and building the batching would not have helped, because a
window only coarsens cadence when it holds more than one order and this venue
produces about 1.6 a day. The scope rule above still put the row in the matrix
correctly, since *when we write* genuinely is ours. **What the rule cannot tell
you is whether owning a leak means you can close it**, and R16 is the row where
that gap showed up. `docs/market-invariants.md` INV-18.

**R12 and R14 are the two that change the design rather than the document.** R12
because the zero-fork settlement path discloses the pair by construction, so
either the matrix admits that cell is `(full, all, immediate)` and we say so, or
D-08's order-book-only layer 2 moves up the roadmap. R14 because it is answered
by *not writing something*, which is a constraint that has to reach the storage
layout before code, not after.

## Out of the matrix, stated as residuals

| # | Residual | Why no cell can hold it | Where it is stated |
|---|---|---|---|
| X1 | **issuer-scoped aggregates.** Subscription and redemption pressure per share class | a property of the fund, not of any participant. A per-trader matrix has no index for it | `SECURITY-MODEL.md` §6.2 out of scope |
| X2 | **submission routing.** Which node saw it first | a property of the transport. Exists whatever the venue writes on-chain | D-15, and §6.4 |
| X3 | **aggregate hiding quota.** MiFIR Art 5's volume cap suspends a mechanism by market share | second-order: a constraint on how much of the market may use a cell, not on any cell | MK-015, and D11 |

Naming these is not a weakness. A matrix that silently omitted them would be
claiming completeness it does not have, and a reviewer who found X1 unaided
would be right to discount everything else.

---

## The cost check, which discharges AB-033 probe (b)

`[DERIVED]` against measured constants: 6,150 gas per Groth16 public input
(spike 1), 214,202 gas to verify, 15,000,000 gas per contract call (AB-031).

| | inputs | gas | share of cap |
|---|---|---|---|
| F4's ten primitives | 10 | 275,702 | 1.8% |
| **the seventeen-row set** | 17 | **318,752** | **2.1%** |
| headroom to the cap | ~2,100 | 15,000,000 | 100% |

Adding seven derived rows costs **43,050 gas, 0.29 percent of one call.** The
scope question is not gas-constrained and never was. Anyone arguing a row out of scope
on cost is arguing from an intuition the measurement contradicts.

---

## Two structural corrections D9 must make, not inherit

Both are inherited assumptions that the venue work falsified. Stating them is
cheap; discovering them mid-build is not.

**1. `G x O x T` is not a clean product.** `[SOURCE]` AB-033. Under conditional
disclosure, the recipient set is computed from the hidden values, so `O` depends
on `G`. DISCLOSURE-LATTICE defines `O` as a powerset the discloser chooses. R13's
cell cannot be written in the current notation. D9 either extends the notation or
writes R13 as a special case and says which.

**2. The matrix presumes the venue can choose a disclosure per row at all.**
`[SOURCE]` AB-034 plus [μ1.4](../mesh/consensus-visibility.md). Only branch C
supports that; on branches A and B the disclosure is set by where the book lives,
not by a policy. D9 must state this as its own precondition rather than assume
it, because it is the assumption that makes the whole artifact meaningful.

---

## What is now unblocked

D9 can open. Its inputs are fixed: seventeen rows, three named residuals, two
stated preconditions, and a cost model showing the row count is free. The
remaining question is genuinely the one D9 was written to answer, which is what
goes in each cell.

**Carried forward as work, not as doubt:** R12 has no mechanism yet and it is the
row most likely to move D-08 up the roadmap. That is the first cell to fill.

## Sources

- In-repo `[MEASURED]`: `marketplace/probes/derived-rows.out`,
  `probes/consensus-observers.out`, `spikes/bn254/`, AB-031
- `marketplace/EVIDENCE.md` MK-008, MK-009, MK-011, MK-013, MK-015, MK-020
- `AUDIT-BOX.md` AB-023, AB-031, AB-033, AB-034, AB-035
- `mesh/consensus-visibility.md` C-03, C-05
- `privacy-abstraction/DISCLOSURE-LATTICE.md` §I.2, §II
