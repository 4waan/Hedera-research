# Marketplace foundation

The evidence base for **F3b** (six venues) and **D10**
(Hedera's own record), which are the two blocks that feed **D9**, the disclosure
matrix.

**Reads:** [STUDY_PLAN.md](../STUDY_PLAN.md) F3b, F4, D9, D10, and
[privacy-abstraction/DISCLOSURE-LATTICE.md](../privacy-abstraction/DISCLOSURE-LATTICE.md)
for the coordinate system. Read the lattice first. Without it there is nowhere to
put a fact.

**Writes:** [EVIDENCE.md](EVIDENCE.md), the ledger. Venue dossiers in
[venues/](venues/). Reproducible probes in [probes/](probes/). Design-affecting
findings cross-file to the audit box.

---

## The filter

The hardest part of this block is not finding material. It is that almost
everything written about market structure is *about* markets rather than *usable
in* our matrix, and a folder that accepts it becomes a library nobody reads.

So the folder is not a library. It is a **queue of cell edits**. The question it
exists to answer is one question, asked of every source:

> For each of F4's nine data items, what did a real venue actually disclose, to
> whom, until when, and by which mechanism? And what did it still prove while
> that information was hidden?

### The admission test

A fact enters only if all four have answers. Three out of four is not a partial
pass, it is a rejection, because a fact missing any one of these cannot be
written as a cell.

| | Question | Rejected if |
|---|---|---|
| **1** | **Which row?** Which of F4's nine items does it touch: trader identity, KYC credential, order size, order price, execution price, quantity, asset, compliance validity, settlement validity? | It describes the venue rather than a data item. "Venue X is private" names no row and enters nothing. |
| **2** | **Which coordinate?** Does it resolve to a point in `G x O x T`: a granularity, a set of observers, a time? | It resolves to a mood. "Institutional-grade confidentiality" is not a coordinate. |
| **3** | **Which mechanism?** Permissioning, delay, aggregation, cryptography, or **conditional disclosure**. | It is a *sixth* mechanism. Same rule as before: not a rejection, it is F3b's falsification condition and the most valuable fact available. Log it first, in bold. |
| **4** | **Who is trusted?** For that disclosure policy to hold, who has to be honest? | Nobody can say. An unanswerable trust question means the source is marketing. |

Then a verdict against the D9 starting position in
[DISCLOSURE-LATTICE.md](../privacy-abstraction/DISCLOSURE-LATTICE.md) II:

| Verdict | Meaning |
|---|---|
| `SUPPORTS` | A real venue puts that row at that coordinate. Moves a cell from opinion to cited opinion. |
| `REFINES` | The coordinate is right, its shape is wrong. Usually: we assumed one constant and the market uses a function. |
| `COLLIDES` | A real venue puts the row somewhere our matrix says it should not be, or cannot express. Changes a design decision. |
| `PARKED` | Real, load-bearing for someone, not for us on a twelve-day clock. One line, a named source, and it stops there. |

### Collisions rank above support, and are logged first

A `SUPPORTS` row makes the pitch better. A `COLLIDES` row makes the *design*
better, and it is the only kind of finding that can still change what we build.
The instinct to collect confirmations is the failure mode of this block, so the
ledger is ordered by verdict, collisions at the top, and a collision is never
softened into a refinement to keep the matrix tidy.

The same asymmetry applies to `unknown`. F3b already rules that an honest
`unknown` beats a plausible cell, and that `unknown` on a permissioned venue *is*
the finding: it means the disclosure policy was never a published property of the
system, only a consequence of the guest list.

### What is excluded, deliberately

Written down so it does not get re-litigated every sitting.

- **Market size, growth, and importance.** They argue the market matters. Nobody
  disputes that. They fill no cell.
- **Technology stack facts that are not disclosure decisions.** Which database,
  which consensus, which cloud. Only relevant where it decides *who can see*.
- **Secondary commentary where a primary source exists.** A rule number, an
  article number, a message field, or a function. F3b's standard, and it is what
  makes a row survive a judge who knows the domain.
- **Anything that becomes a research project.** Curve maths, MEV mechanism
  design, the full MiFIR text. One `PARKED` line, then stop.

> **The fifth mechanism was found, in V3.** Question 3 originally listed four.
> Turquoise Plato Block Discovery hides a block order by none of them: it
> releases a predicate computed over the hidden data, to a recipient set computed
> from the same data, at the moment the predicate turns true. See
> [MK-013](EVIDENCE.md) and `AB-033`. The test still works the same way, and the
> list is now five with the same standing invitation to break it.

### The one thing that closes a row

Per Method rule 1, reading does not close anything. A venue row closes when its
nine cells are filled from a primary source and its trust question is answered.
A Hedera row closes when the disclosure is **observed on the ledger**, not when
the press release is read. Probes that do this live in [probes/](probes/) and are
runnable by someone who does not trust us.

---

## Where the block lands

All six venue rows are filled. The cross-venue read, which is `OUTPUT F3b`, is
in [SIX-VENUES.md](SIX-VENUES.md): the matrix, the three cells every venue
hides, the three none hide, and the empty quadrant we are claiming.

## Status

| Row | Subject | State | Closed by |
|---|---|---|---|
| V1 | OTC bond markets: RFQ plus the public tape | **filled** | FIX 1171/1172, MiFIR Art 11(3), RTS 2 categories, FINRA 6750 |
| V2 | Kinexys (JPMorgan) | **filled, mostly `unknown` by construction** | Primary product pages, and their silence |
| V3 | Dark pools: the waiver rulebook | **filled, and it falsified the taxonomy** | MiFIR Art 4(1) waivers and Art 5 cap, plus the Turquoise Plato Block Discovery service description |
| V4 | dYdX v4 and Hyperliquid: where the book sits | **filled, branched, and it decided an architecture** | [probes/hyperliquid-positions.sh](probes/hyperliquid-positions.sh) against the live API, plus dYdX architecture docs |
| V5 | Uniswap: total transparency priced | **filled** | v4 flash accounting, plus Qin/Zhou/Gervais IEEE S&P 2022 |
| V6 | Polymarket: the aggregate as the product | **filled** | `OrderStructs.sol` and `docs/Overview.md`, read from the repository |
| H1 | Archax / abrdn / Lloyds on Hedera mainnet | **probed, and it produced the strongest finding in the folder** | [probes/hedera-register.sh](probes/hedera-register.sh) |
| H2 | ATS shipped defaults | covered by [mesh/transfer-path.md](../mesh/transfer-path.md) | already done |
| H3 | Remaining Hedera institutional deployments | open | Woolard report, taskforce, Council member deployments |

V1 and V2 were the two the plan required before Sept 4, because they are the two
that change what we build. They are done. V3 to V6 are one sitting each and can
run in dead calendar time.
