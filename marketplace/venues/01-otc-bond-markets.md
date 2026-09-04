# V1. OTC bond markets: RFQ plus the public tape

**State:** filled. **Closed by:** FIX 5.0 SP2 tags 1171 and 1172; MiFIR Article
11(3) and ESMA Decision ESMA74-276584410-11245; RTS 2 as amended by Delegated
Regulation (EU) 2025/1246; FINRA Rules 6750 and 6730.

The closest analogue to a repo venue, and the only one of the six where the
disclosure policy is written down as enforceable rules rather than inferred from
behaviour. It is therefore the row that carries the most weight, and it is filled
first.

Treat it as **two venues stacked**: a bilateral negotiation layer (RFQ, governed
by message format and counterparty selection) and a publication layer (the tape,
governed by regulation). They hide different things by different mechanisms, and
conflating them is how this row would go wrong.

---

## The nine rows

`G` granularity, `O` observers, `T` time. Blank cells are not applicable rather
than unknown.

### Pre-trade: the RFQ layer

| F4 row | G | O | T | Mechanism |
|---|---|---|---|---|
| Trader identity | full | the dealers on the request | at request | permissioning (`RespondentType`) |
| KYC credential | full | the dealer, out of band | pre-relationship | permissioning |
| Order size | full, or absent for a market-style quote | selected respondents only | at request | permissioning |
| Order price | full (the quote) | selected respondents only | at quote | permissioning |
| Asset | full | selected respondents only | at request | permissioning |

The dial is `RespondentType` (tag 1172): `1` all market participants, `2`
specified market participants, `3` all market makers, `4` primary market
maker(s). `PrivateQuote` (tag 1171) is the boolean above it: public, meaning
available to the market, or private, meaning available to a specified
counterparty only. Omitting `OrderQty` (38) and `Side` (54) yields a market-style
two-way quote, which is how a requester asks a price without disclosing size or
direction. That is a granularity move, in the message, chosen per request.

**So the pre-trade privacy dial is three fields wide** and every position on it
is a point in `O x G`. See [MK-004](../EVIDENCE.md#mk-004--the-observer-axis-is-already-a-wire-field-and-has-been-for-two-decades).

### Post-trade: the tape

EU, bonds, under the reviewed MiFIR:

| F4 row | G | O | T | Mechanism |
|---|---|---|---|---|
| Execution price | full | all | 15 min, or T+1 / T+2 by category | delay |
| Quantity | full | all | 15 min, EOD, or up to 2 weeks by category | delay |
| Asset | full | all | with the print | none |
| Trader identity | none | none | never | omitted from the tape entirely |
| Settlement validity | not published | | | |

Five categories, liquid 1/3/5 and illiquid 2/4/5, flags `MLF1` `MIF2` `LLF3`
`LIF4` `VLF5` `VIF5`, three deferral thresholds per bond, category assigned on
bond type, issuance size, trade size and liquidity. For Group 1 Category 1
sovereigns, ESMA's February 2026 decision permits omitting **volume only**, with
volume published by end of day and price on the ordinary clock.

US, corporate bonds, under TRACE:

| F4 row | G | O | T | Mechanism |
|---|---|---|---|---|
| Execution price | full | all | immediate on receipt, 15 min outer limit to report | delay |
| Quantity | capped: `5MM+` / `1MM+` above the cap | all | immediate | **aggregation** |
| Quantity, again | full | all | +6 months after quarter end, in a separate data product | delay |
| Trader identity | none | none | never | omitted |

Plus 6750(d)'s never-disseminated list, 6750(c)'s end-of-day treatment of
on-the-run Treasury coupons, and 6750(b)'s weekly and monthly aggregation for
CMOs at or above $1m.

---

## The two questions

**What is hidden, from whom, until when, by which mechanism?**

Pre-trade, everything is hidden from everyone outside the chosen respondent list,
by permissioning, until the requester chooses otherwise. Post-trade, identity is
hidden permanently and unconditionally, price and size are hidden temporarily by
delay, and size is additionally hidden permanently in resolution by the TRACE cap
while remaining recoverable in full six months later.

**What does the venue still prove while that information is hidden, and who has
to be trusted?**

Almost nothing, and the answer is the point of this row. The tape is an
*obligation*, not a proof. Its integrity rests on the reporting firm, its APA or
TRACE, and the supervisor's enforcement. Nothing published is verifiable against
anything: a reader cannot check that a print corresponds to a real trade, that
the deferral category was assigned honestly, or that a `5MM+` flag is not
concealing a differently sized trade. Trusted parties are the executing firm, the
APA, and the NCA or FINRA.

**This is the cleanest available statement of what our design adds.** The
disclosure policy in this market is already articulate, already graded, already
machine-readable on the wire, and already regulator-written. It is simply
unverified. We are not proposing a privacy model the market lacks. We are
proposing a proof that the model it already has was applied honestly.

---

## What this row changed

- [MK-001](../EVIDENCE.md#mk-001--price-and-volume-of-the-same-trade-run-on-two-different-clocks), collision: delta is per row, not per venue.
- [MK-002](../EVIDENCE.md#mk-002--a-cell-has-to-be-a-trajectory-not-a-point), collision: a cell is a trajectory.
- [MK-003](../EVIDENCE.md#mk-003--the-block-size-threshold-is-a-function-of-two-variables-with-three-breakpoints), refinement: the threshold has two variables and three breakpoints.
- MK-004, MK-005, MK-006, MK-007, support.

**Four mechanisms, and the taxonomy held.** Permissioning (respondent lists),
delay (deferrals), aggregation (TRACE caps, CMO weekly prints, Notice 25-17
allocation aggregation) all appear. Cryptography does not appear anywhere in this
row, which is F3b's hypothesis surviving its first real test. No fifth mechanism
was found.
