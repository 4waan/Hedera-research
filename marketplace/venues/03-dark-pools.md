# V3. Dark pools: the waiver rulebook

**Status: filled. It falsified F3b's four-mechanism hypothesis.**

**Primary sources.** MiFIR (Regulation (EU) No 600/2014) Article 4(1) and
Article 5, read on EUR-Lex. *Turquoise Plato Block Discovery Trading Service
Description*, v2.28.1, London Stock Exchange Group, 40pp, read in full from
`docs.londonstockexchange.com`. The Turquoise document is the venue rulebook the
plan asked for, and it is unusually good: it specifies field by field what each
party learns at each step.

Every phrase quoted below is checkable without trusting this repository:
[probes/turquoise-quotes.sh](../probes/turquoise-quotes.sh) refetches the PDF
from LSEG and asserts each one appears verbatim. It reports 10 of 10, and its
output is in [turquoise-quotes.out](../probes/turquoise-quotes.out).

`G` granularity, `O` observers, `T` time. Blank cells are not applicable rather
than unknown.

---

## 1. The four waivers, as four positions in the lattice

Article 4(1) is a closed list. Each point is a different axis move, which is why
it is worth quoting rather than paraphrasing.

| Waiver | MiFIR Art 4(1) text | Axis it moves |
|---|---|---|
| **Reference price** | "systems matching orders based on a trading methodology by which the price of the financial instrument [...] is derived from the trading venue where that financial instrument was first admitted to trading or the most relevant market in terms of liquidity, where that reference price is widely published and is regarded by market participants as a reliable reference price" | `G` on order price, to `⊥`, by **not having a price of its own** |
| **Negotiated transaction** | "systems that formalise negotiated transactions" made "within the current volume weighted spread" (b)(i), in an illiquid instrument (b)(ii), or "subject to conditions other than the current market price" (b)(iii) | `O`, to the negotiating pair |
| **Large in scale** | "orders that are large in scale compared with normal market size" | `G` on order size, gated on a threshold |
| **Order management facility** | "orders held in an order management facility of the trading venue pending disclosure" | `G` on quantity, released on execution rather than on a clock |

The third is the one D9 already models: the LIS ramp of DISCLOSURE-LATTICE
III.3, in production, as a named legal instrument. The fourth is the iceberg
order, and it is the first thing in this study that the four-mechanism taxonomy
does not have a name for. "Pending disclosure" is not delay: nothing in the rule
says *when*. The reserve is revealed as it executes, so the release is triggered
by the data, not by a clock.

---

## 2. Turquoise Plato Block Discovery, pre-trade

The service runs on the reference price and LIS waivers. A participant submits a
**Block Indication**, defined as "a non-actionable indication of interest [...]
that corresponds to a live In-hand Order over which the Participant has full
discretion, and can be immediately converted into a firm QBO if a matching
opportunity is identified."

Two observers have to be tracked separately, and the whole row turns on the
difference: **Turquoise** as operator, and **another participant**.

| F4 row | G to the operator | G to a matched counterparty | G to everyone else | O | T | Mechanism |
|---|---|---|---|---|---|---|
| Trader identity | full | `⊥` | `⊥` | operator | at BI | conditional |
| KYC credential | membership | `⊥` | `⊥` | operator | pre-relationship | permissioning |
| Order size | full | **one bit**: "at least your own MES" | `⊥` | operator, then the matched pair | at BI, then at match | **conditional** |
| Order price | full | `⊥` | `⊥` | operator | at BI | **conditional** |
| Execution price | not set by either side | midpoint of the primary BBO | published post-trade | all | t₀ + deferral | reference pricing |
| Quantity | full | `⊥` until the uncross | published post-trade | all | t₀ + deferral | conditional, then delay |
| Asset | full | full | `⊥` pre-trade | operator, then the pair | at BI | conditional |
| Compliance validity | membership | assumed | `⊥` | operator | pre-relationship | permissioning |
| Settlement validity | CCP | CCP | `⊥` | operator | post-trade | permissioning |

The event model states the pre-trade position twice, in the venue's own words:
at step 1, "No information sent to any other Participant." At step 2, on a
match, "Parties submitting BIs that match each receive an OSR (but no details
regarding nature of counterparty Order/BI)."

### The cell that matters

The Order Submission Request is specified to the field. Section 5.3.4:

> "The OSR contains no information about the size, MES or Price of the potential
> counterparty, but receipt of an OSR implies that a potential match is available
> for at least the Participant's own MES."

That is a disclosure of **exactly one predicate over hidden data**, released to
**exactly the parties the predicate names**, at **the moment the predicate turns
true**. It is the output shape of a ZK circuit, written into a production venue
rulebook in 2016, and achieved by having an operator who reads the cleartext.

---

## 3. The fifth mechanism

F3b's hypothesis is that every venue that hides anything hides it by
permissioning, delay, aggregation, or cryptography. Block Discovery hides a
block order by none of them:

- Not **permissioning**. Participation is opt-in and open to all Turquoise
  participants. Nobody is outside a garden; the information is withheld from
  everyone equally, including the people who are inside.
- Not **delay**. The BI is never published at all, and the OSR fires on a match,
  not on a clock. There is no δ to name.
- Not **aggregation**. No total is published in place of the line item.
- Not **cryptography**. The operator sees every field in the clear.

The mechanism is **conditional disclosure**: release a predicate computed over
the hidden data, to a recipient set computed from the same data. Article 4(1)(d)
is the same mechanism in the iceberg case. This is the falsification the plan
asked for, and per the plan's own instruction it is logged before anything that
supports us.

**It does not weaken P3. It sharpens it.** Conditional disclosure is not a rival
to cryptography; it is the thing cryptography implements without an operator.
The two mechanisms produce the same disclosure and differ only in who must be
trusted to compute the predicate honestly. That reframes the claim: we are not
introducing a disclosure shape the market lacks, we are removing the operator
from a disclosure shape the market already runs on.

---

## 4. What the venue proves, and who must be trusted

The answer is unusually explicit, because Turquoise had to solve our exact
problem: a participant indicates a block, gets told a match exists, and must
then honour the indication. Nothing forces them to. The venue's answer is
**Reputational Scoring**, not a proof.

- A valid firm order in response scores 50 to 100, "depending on the size of the
  QBO relative to the original BI".
- "Failure to send a valid QBO meeting the above criteria results in a
  zero-score."
- Events are combined by recency: "The most recent event has a weighting of 100,
  the next most recent 99, and so on."
- "If a user's Composite Reputational Score drops below the specified
  Reputational Score Threshold at any time, the user will be immediately
  excluded from further use of the service."
- The score travels back to the participant on FIX tag **27012**, on every OSR
  for a matched BI.

Read against P3, that is the whole argument in one artifact. The market has a
production mechanism for conditional disclosure, and its enforcement is
**retrospective, probabilistic and punitive**: it detects dishonesty after the
fact, scores it, and eventually ejects you. A proof is **prospective,
deterministic and preventive**: the dishonest order cannot be submitted. The gap
between those two sentences is the product.

Who must be trusted: LSEG, completely, for the predicate and for the score.
There is a second trusted party the design cannot avoid, and it is the reason
the volume cap exists.

---

## 5. The volume cap, which is not a cell at all

Article 5(1), original text: "In order to avoid any negative impact on the price
formation process, competent authorities shall apply a volume cap mechanism for
orders placed in systems which are based on a trading methodology by which the
price is determined in accordance with Article 4(1)(a)".

The reference price waiver hides a price by **not producing one**. The midpoint
is imported from the lit market. So dark trading under this waiver consumes a
public good it does not contribute to, and if enough volume moves dark, the
reference price the whole mechanism depends on degrades. The cap is the
regulator metering that consumption.

Nothing in `G × O × T` can express it. A cap is not a coordinate on a cell; it
is a **quota on how much volume may be assigned to a cell**, measured across the
market and enforced by suspension. It is second-order, and D9 has no
representation for it.

This matters for us specifically, because our design has the same free-riding
property whenever execution price is deferred. See MK-015.

---

## 6. The two questions

**What is hidden, from whom, until when, by which mechanism?** Pre-trade,
everything except the asset is hidden from every participant, including the
eventual counterparty, until a match exists. On a match, exactly one bit crosses
to exactly two parties. Post-trade the trade joins the tape under the same
deferral regime as V1. The mechanism is conditional disclosure, with
permissioning underneath it and delay on top.

**What does the venue still prove, and who is trusted?** It proves nothing. It
scores. Every guarantee in the service rests on LSEG evaluating the match
predicate honestly on data only LSEG can see, and on a recency-weighted heuristic
standing in for the enforcement that would otherwise be needed.

---

## Honest limits

- Article 5 was amended by Regulation (EU) 2024/791, which replaced the double
  volume cap with a single cap. The verbatim text above is the original. The
  amended figure (7%, applied by venues rather than competent authorities) comes
  from a secondary source and **must be replaced with the consolidated primary
  text before the pitch**. The finding in §5 does not depend on the number: a
  cap exists either way, and it is the existence that D9 cannot represent.
- Turquoise is one venue. The conditional-disclosure mechanism is general to
  conditional order types across dark venues, but this row cites one rulebook.
- Post-trade cells are inherited from V1 rather than re-derived, since the
  deferral regime is the same one.
- No claim is made about the volume cap's effect. That is a market-structure
  research question and is parked.
