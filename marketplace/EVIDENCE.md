# Evidence ledger

Append-only. Every entry passed the four-question admission test in
[README.md](README.md). Ordered by verdict: **collisions first**, because they
are the only findings that can still change what we build.

Coordinates are written `(G, O, T)` per
[DISCLOSURE-LATTICE.md](../privacy-abstraction/DISCLOSURE-LATTICE.md) I.3.

---

# COLLIDES

## MK-001 · price and volume of the same trade run on two different clocks
**Row:** execution price, quantity · **Mechanism:** delay, applied per field
**Source:** ESMA Board of Supervisors, *Decision on allowing supplementary
deferrals for sovereign bonds under MiFIR*, ESMA74-276584410-11245, adopted
17 February 2026, Articles 1 to 3. Made under MiFIR Article 11(3)(a). Effective
**4 May 2026**.

Article 1 scopes it to sovereign bonds in **Group 1, Category 1 (medium size,
liquid)** as defined in Table 2.6 of Annex III of RTS 2 (Delegated Regulation
(EU) 2017/583). Article 2 then grants a supplementary deferral

> "limited to the omission of the volume of individual transactions"

with Article 2(2) requiring that volume be published "by the end of the trading
day." The price is not deferred at all: it goes out on the ordinary clock, which
recital (4) confirms is 15 minutes. ESMA separately raised the *price* deferral
to T+1 and T+2 for Categories 3 and 4 while volume deferrals for those run to two
weeks.

**Source quality, stated so it is not overcited.** The ESMA decision itself was
read in full and every quoted phrase above comes from it. The Category 3 and 4
price deferrals (T+1, T+2) and the two-week ETC/ETN volume deferral come from
summaries of ESMA's December 2024 Final Report, not from the Report or from the
RTS 2 Annex II table directly. **Before the pitch, pull the Annex II table and
replace those two figures with the primary text.** The collision below does not
depend on them: it stands on the ESMA decision alone.

**Coordinate.** Execution price `(full, all, t0+15min)`. Quantity of the same
trade `(bottom, all, t0)` then `(full, all, end of day)`.

**Why it collides.** DISCLOSURE-LATTICE II gives execution price and quantity the
same `t0+delta`. The regulator uses two different deltas for the same trade, and
the instrument class where it does so is sovereign bonds, which is our
collateral. **Delta is not a venue constant. It is a per-row constant.**

**Consequence.** D9's "pre-trade vs post-trade split becomes the venue's state
machine phases" is too coarse. Either the state machine gets one publication
phase per field, or the matrix carries a per-row release time and the state
machine is driven by a schedule rather than by phases. Cheap to fix now,
expensive after the orderbook storage layout is written.

---

## MK-002 · a cell has to be a trajectory, not a point
**Row:** quantity · **Mechanism:** aggregation composed with delay
**Source:** FINRA Rule 6750, dissemination caps. $5,000,000 par for investment
grade and $1,000,000 par for non-investment grade, disseminated as `5MM+` and
`1MM+`. The uncapped size is released in the Historic Corporate Bond Data Set six
months after the calendar quarter in which the trade was reported.

**Coordinate.** Quantity is `(one bit above the cap, all, t0)` **and**
`(full, all, t0 + ~6 to 9 months)`. One row, two points, and the row is not
finished at either of them.

**Why it collides.** This is D9a defect 2 in a second guise. Defect 2 caught the
time axis being collapsed into the observer axis. This catches something the
corrected lattice still cannot say: a cell holding a *trajectory* rather than a
point. TRACE never delays quantity, it coarsens it, permanently, and separately
publishes the exact figure much later to a different audience in a different
product.

**The fix is one sentence and it costs nothing.** A cell is a monotone function
`T -> G x O`, not an element of `G x O x T`. The product lattice already supports
this: monotone functions into a complete lattice form a complete lattice
pointwise. Nothing in the algebra changes, and "coarse now, exact later" becomes
expressible.

**And it is a join to check.** The `5MM+` bit at t0, joined with the exact size at
t0 plus six months, joined with what the counterparty already knew, is exactly the
k-one-bit arithmetic in DISCLOSURE-LATTICE II. Add "the same observer at two
points on one row's trajectory" to the list of subsets checked.

---

## MK-013 · a fifth mechanism exists, and F3b's taxonomy is wrong

**Row:** order size, order price, quantity, pre-trade · **Mechanism:** none of
the four

**Source.** *Turquoise Plato Block Discovery Trading Service Description*
v2.28.1, London Stock Exchange Group, sections 5.2 and 5.3.4, read in full. Plus
MiFIR Article 4(1)(d), "orders held in an order management facility of the
trading venue pending disclosure".

Turquoise Plato Block Discovery hides a block order from every other
participant, then releases exactly one fact to exactly two of them. Section
5.3.4:

> "The OSR contains no information about the size, MES or Price of the potential
> counterparty, but receipt of an OSR implies that a potential match is available
> for at least the Participant's own MES."

It is not permissioning: participation is opt-in and open, and the order is
hidden from insiders too. It is not delay: the indication is never published and
the release fires on a match, so there is no delta to name. It is not
aggregation: no total substitutes for the line item. It is not cryptography: the
operator reads every field in the clear.

The mechanism is **conditional disclosure**: publish a predicate computed over
the hidden data, to a recipient set computed from the same data, at the moment
the predicate becomes true. Article 4(1)(d) is the same mechanism in the iceberg
case, where the reserve is released by execution rather than by a clock.

**What it changes.** F3b's hypothesis is falsified, which the plan correctly
called the better outcome. P3's uniqueness claim has to be restated, and the
restatement is stronger than the original. Conditional disclosure is not a rival
to cryptography, it is what cryptography implements without an operator: the two
produce the same disclosure and differ only in who must be trusted to evaluate
the predicate. So the claim is no longer "only cryptography can hide this while
keeping compliance checkable", which was always vulnerable to exactly this
counterexample. It is "the market already runs on this disclosure shape and pays
for it with a trusted evaluator, and we remove the evaluator."

---

## MK-014 · the observer set is computed from the data, and the thing disclosed is not one of F4's nine rows

**Row:** all of them, and a tenth that does not exist · **Mechanism:**
conditional disclosure

**Source.** As MK-013, against DISCLOSURE-LATTICE line 75, which defines the
observer axis as "Powerset of principals under inclusion; joins are unions."

Two structural problems, both from the same paragraph of the rulebook.

**First, `O` is not chosen by the discloser.** In the lattice, a cell names a set
of principals. In Block Discovery the recipient set is whoever's minimum
execution size the hidden order happens to satisfy, so `O` is a function of the
values sitting in the other rows. The three axes of `G x O x T` are not
independent, and the product structure quietly assumed they were.

**Second, and worse, the disclosed datum is not in F4.** What crosses to the
counterparty is not order size, it is the predicate "a match exists at or above
your MES". F4's nine rows are all primitive fields of a trade. This is a derived
fact over them, and the matrix has no row it can go in.

**That is the third independent sighting of the same defect.** MK-009 found the
public `GRANTED -> REVOKED` transition, which is credential history, not the
credential. MK-011 found counterparty relationships, which are a fact about
pairs of trades. Now a match predicate. **F4 enumerates primitive fields, and
the leaks that actually matter are derived facts over them.**

**The fix, which is cheap.** Admit derived rows: a row may be a function `f` of
other rows, and it carries its own `(G, O, T)` coordinate like any other. The
prior art is already in the folder. The lattice doc's own comparison table notes
BBS+ does selective disclosure "plus predicates", and data-dependent
declassification is exactly what delimited release and abstract non-interference
were built for, both already cited in section IV. Cost on our side is one public
input, 6,150 gas against a 15,000,000 cap, so this is not gas-constrained
either.

**Open, and it belongs to D9, not here:** which derived rows are in scope. At
minimum the match predicate, credential history, and counterparty relationship.
There is no principled stopping point yet, and inventing one is the D9 block's
job.

---

## MK-015 · there is a regulatory cap on aggregate hiding, and it is not a coordinate

**Row:** none · **Mechanism:** a quota over mechanisms

**Source.** MiFIR Article 5(1), original text: "In order to avoid any negative
impact on the price formation process, competent authorities shall apply a volume
cap mechanism for orders placed in systems which are based on a trading
methodology by which the price is determined in accordance with Article
4(1)(a)".

The reference price waiver hides a price by not producing one: the midpoint is
imported from the lit market. Dark trading under it consumes a public good it
does not contribute to, and past some share of volume the reference price it
depends on degrades. The cap meters that consumption, by suspending the waiver.

Nothing in `G x O x T` can say this. A cap is not a coordinate on a cell. It is a
quota on how much volume may be assigned to a cell, measured market-wide and
enforced by withdrawal of the mechanism. It is second-order, and D9 has no
representation for it.

**Why this is ours and not a curiosity.** Our design has the same free-riding
property wherever execution price is deferred: during the deferral the trade
takes a price signal from the market without contributing one. The first
structural objection a regulator raises to universal per-trade privacy is not
"what do you hide", which D9 answers completely. It is "what fraction of the
market may hide at all", which D9 does not answer.

There is a real answer available and it needs to be written down rather than
improvised at the pitch: a deferral contributes the price eventually, whereas
the reference price waiver never contributes one. Deferred disclosure is a loan
against price formation; the reference price waiver is a permanent
externality. That distinction is the reason a cap that exists for one need not
apply to the other, and it should be stated in D9 as an explicit position.

**Source quality.** The verbatim text above is the original Article 5. It was
amended by Regulation (EU) 2024/791, which replaced the double volume cap with a
single cap applied by venues. The amended figure reached us through a search
summary, not the consolidated text, so it is not quoted here. **Pull the
consolidated Article 5 before the pitch.** The finding does not depend on the
number: a cap exists under both versions, and it is the existence D9 cannot
represent.

---

## MK-018 · the highest-revenue venue in crypto discloses everything

**Row:** all nine · **Mechanism:** none, anywhere

**Source.** Measured, not cited:
[probes/hyperliquid-positions.sh](probes/hyperliquid-positions.sh) section 4,
reading the DefiLlama fees API on the same run that read the positions.

30-day revenue retained: Tether 478.3m, Circle 191.1m, **Hyperliquid Perps
49.1m**, Canton 48.8m, pump.fun 35.9m. The two above Hyperliquid are stablecoin
issuers earning on reserves rather than venues. Among trading venues Hyperliquid
is first by roughly three times, with 1.18bn all-time, and it outearns every
chain listed including Tron. Its nine F4 rows are all `full`, to `all`,
immediately, by no mechanism.

**Stated in its strongest form, because managing it is worse than facing it:**
the most commercially successful trading venue in crypto hides nothing, and a
privacy layer is not what made anyone rich.

**The resolution, which is a real one.** Transparency is affordable exactly
where the leaked information is cheap. A perp position sits in a liquid,
continuously traded, globally fungible instrument, and being read costs basis
points. A block in one illiquid bond is the opposite case, which is the entire
reason the LIS waiver exists and why every institutional venue in V1, V2 and V3
hides. The evidence is negative and checkable: **there is no institutional bond
or repo activity on Hyperliquid.** The absence of our customer from the most
successful venue in the space is the datum.

Nor is the leak free even there. See MK-021.

**What it changes for us.** The pitch must not claim that transparency does not
work. It manifestly does, at scale, profitably. The claim is narrower and
survives contact: transparency is priced by the liquidity of what leaks, and the
instruments Hedera has already onboarded sit at the expensive end. Any framing
that needs Hyperliquid to be a failure is a framing that loses the room.

---

## MK-019 · in a consensus-held book, disclosure is not a policy, it is a consequence of replication

**Row:** order size, order price, pre-trade · **Mechanism:** none available

**Source.** Hyperliquid architecture (HyperCore holds the central limit order
book as consensus state under HyperBFT) against dYdX Chain architecture
documentation (validators "store orders in an in-memory orderbook (i.e. off
chain and not committed to consensus)", gossip them, and commit only executed
trades).

Byzantine fault tolerance over an order book requires every validator to hold
the same book, and a validator holding the book *is* the disclosure. On
Hyperliquid the public API leaks nothing that consensus had not already
required. **Privacy cannot be added to that design, because the leak is the
replication.**

dYdX does not solve this, it relocates it. The book still sits in cleartext in
every validator's memory and simply never reaches consensus, so the public sees
only fills. That is a public leak converted into a leak to a fixed operator set,
which is permissioning, on a network that presents as decentralised. It costs
one thing Hyperliquid does not pay: since resting orders are never committed,
**no record exists against which anyone can later check that matching was
honest.**

**What it changes.** The question "where does the book live" is not an
implementation detail downstream of the disclosure matrix, it is upstream of it.
Branch A cannot express a disclosure policy at all; branch B can only express
one about the public. Only committing a hidden book restores per-row choice.
D9's matrix silently assumes a venue that *can* choose, and exactly one of the
three architectures can. That assumption should be written down rather than
inherited.

---

## MK-020 · the most dangerous field on a live venue is a derived one, and F4 has no row for it

**Row:** none of the nine · **Mechanism:** none

**Source.** `api.hyperliquid.xyz/info`, `clearinghouseState`, in the probe.

Every open position returns `liquidationPx`, the price at which the account is
forcibly closed, to eleven significant figures, unauthenticated. One live
example from the run: an account worth 81.9m short 1,707 BTC at entry 70,551.3
with liquidation at 124,904.26132943. Everything needed to size a squeeze
against that account is a field in a public response body, and the register of
44,627 accounts to choose a target from is a separate public endpoint.

`liquidationPx` is a function of size, margin, leverage and mark price. It is
not one of F4's nine rows, and it is by a distance the most sensitive number on
the account.

**Fourth independent sighting of one defect.** MK-009 credential history, MK-011
counterparty relationships, MK-013 the match predicate, and now the liquidation
price. Four venues, four different derived facts, each more sensitive than the
primitive fields it is computed from. MK-014 proposed admitting derived rows as
a cheap fix; after a fourth sighting that is no longer a proposal to consider,
it is a confirmed gap D9 has to close, and the open question is only which
derived rows are in scope.

---

## MK-022 · the time axis needs a point before zero, and it is where the money is lost

**Row:** order size, order price, asset, trader identity · **Mechanism:** none

**Source.** Uniswap v4 swap path over a public mempool. Qin, Zhou and Gervais,
*Quantifying Blockchain Extractable Value: How dark is the forest?*, IEEE S&P
2022 (arXiv 2101.05511).

Every venue in V1 to V4 and V6 discloses at t₀ or later: at the trade, at the
tape, after a deferral. A swap broadcast to a public mempool is disclosed
**before the trade exists**, to anyone running a node, and it may never execute
at all. Size, price bound, asset and sender are public while the trade is still
an intention.

D9's rows are all at t₀ or t₀+δ. The axis needs **t₀ − δ**, and it is the only
point on it where an observer can reorder disclosure and execution. That is
precisely what sandwiching is.

The cost, with a method we can restate: classify each transaction by its
position relative to the transactions it profits from, then price the
difference. Over 32 months the authors attribute **540.54m USD** across 11,289
addresses, 49,691 assets and 60,830 markets, with a largest single instance of
4.1m USD, 616.6 times that block's reward.

**What it changes.** Two things, and the second is ours. First, D9 must carry a
pre-trade time coordinate explicitly, because a matrix whose earliest point is
t₀ cannot express the leak that costs the most. Second, `AB-034` and μ1.4 are the
same question asked twice: whether anything about our order is visible at t₀ − δ
on Hedera. If it is, the disclosure matrix is decided before D9 gets to choose
anything.

**Source quality.** The BEV measurement covers Dec 2018 to Aug 2021. It is used
for the mechanism and the order of magnitude, not as a current figure.

---

# REFINES

## MK-003 · the block-size threshold is a function of two variables, with three breakpoints
**Row:** order size · **Mechanism:** delay, gated on a classification
**Source:** RTS 2 as amended by Delegated Regulation (EU) 2025/1246 of 18 June
2025, deferral flags `MLF1`, `MIF2`, `LLF3`, `LIF4`, `VLF5`, `VIF5`. Five
categories: liquid are 1, 3, 5 and illiquid are 2, 4, 5. Per LSEG TRADEcho's
implementation note, **each bond carries three deferral thresholds**, and
category is assigned on bond type, issuance size, trade size and liquidity
profile. APA production go-live 2 March 2026.

**Why it refines.** M1 requires the threshold to be "a named constant, not a
vibe," and DISCLOSURE-LATTICE III.3 implements it as a monotone ramp in one
variable. The regulator's threshold is a function of `(size, liquidity)` with
three breakpoints, not one of `size` alone.

**The decision this forces, and it belongs in D9.** Either

- the circuit takes a liquidity class as a second public input, which is nearly
  free: one more field element at 6,150 gas against a measured 15,000,000 cap
  (`AUDIT-BOX.md` Measurements), so gas is not the constraint; or
- we state the simplification plainly: we implement the size axis and treat our
  single instrument as one liquidity class.

The second is defensible on a twelve-day clock and the first is cheap. What is
not defensible is shipping a one-variable threshold while claiming to mechanise
MiFIR's, because the difference is one sentence for a judge who knows RTS 2.

---

# SUPPORTS

## MK-004 · the observer axis is already a wire field, and has been for two decades
**Row:** order price, order size (pre-trade) · **Mechanism:** permissioning, at
message granularity
**Source:** FIX 5.0 SP2. Tag **1171 PrivateQuote**, Boolean: "Specifies whether a
quote is public, i.e. available to the market, or private, i.e. available to a
specified counterparty only." `Y` = private, `N` = public. Tag **1172
RespondentType**, int: `1` = All market participants, `2` = Specified market
participants, `3` = All Market Makers, `4` = Primary Market Maker(s). Both used
in `QuoteRequest <R>`; 1171 also in `Quote <S>`, `RFQRequest <AH>`,
`QuoteRequestReject <AG>`.

**Why it supports.** `RespondentType` is an enum over the observer lattice: a
four-point sublattice of the powerset of principals, ordered by inclusion,
exactly DISCLOSURE-LATTICE I.3's `O` axis. F3b predicted that "how many dealers
go on an RFQ *is* the privacy dial." It is not a metaphor. It is
`RespondentType = 2` plus a party list.

**What this changes in the pitch.** We do not have to argue that an observer axis
is a meaningful way to describe disclosure. The market has shipped it since FIX
5.0. We have to argue something narrower and much stronger: **nobody has ever
proved that a message honoured its own disclosure field.**

---

## MK-005 · the disclosure class already travels with the trade, as a public machine-readable flag
**Row:** all published rows · **Mechanism:** delay, declared
**Source:** RTS 2 Annex II deferral flags (above), plus the sovereign
supplementary deferral flags **`OMIS+FULO`** (omit volume, then full volume
later) and **`AGFW+FULG`** (aggregated form, then full granular later), both
supported for customer submission and APA calculation from the 2 March 2026
release.

**Why it supports, and it is the best single sentence in the folder.**
DISCLOSURE-LATTICE III.1 argues on soundness grounds that the disclosure class
*must* be a public input: if the class is hidden, the verifier does not know
which statement is being proved and the prover picks whichever class suits them.
MiFIR arrives at the same requirement for a completely different reason and
publishes the class on the wire next to the trade.

So the architecture's most unusual-looking design constraint is already how the
regulated bond tape works. The gap is not that the class is private. **The gap is
that no one proves the class was applied honestly to the trade underneath it.**
That gap is precisely the circuit, and this is the citation that makes §V's
novelty claim survive being pushed on.

---

## MK-006 · a "never" class exists in production, with the aggregation mechanism written into the rule
**Row:** execution price, quantity, trader identity · **Mechanism:** aggregation
**Source:** FINRA Rule 6750(d): FINRA will **not** disseminate affiliate-principal
transactions, proprietary position transfers connected to mergers and
acquisitions (on three business days' advance notice), List or Fixed Offering
Price Transactions, certain securitized products, most US Treasury securities,
and foreign sovereign debt securities. The same paragraph permits FINRA to
publish aggregated statistics on non-disseminated transactions "without
identifying individual participants or securities."

Also in one rule: 6750(b) publishes CMO transactions at or above $1,000,000 only
weekly and monthly, and aggregated. 6750(c) puts on-the-run nominal Treasury
coupons on end-of-day dissemination. 6750(a) is immediate for everything else.

**Why it supports.** Two things at once. D9's `never` column is not a hackathon
indulgence, it is a live category in the most transparent bond tape in the world.
And F3b's four-mechanism taxonomy does not have to infer aggregation from
behaviour: it is written as a rule, with the re-identification guard ("without
identifying individual participants or securities") stated in the same sentence.

Four points on the T axis inside a single rule is also the cleanest available
demonstration that the axis is real and graded rather than binary.

---

## MK-007 · the time axis has a practical floor, and the regulator just declined to lower it
**Row:** execution price, quantity · **Mechanism:** delay
**Source:** FINRA Regulatory Notice 25-17. FINRA is **not** proceeding with its
proposal to reduce the 15-minute TRACE reporting outer limit under Rule
6730(a)(1) to one minute. The notice's actual amendment, new Supplementary
Material .08 permitting aggregated reporting of allocations across managed
accounts, takes effect 8 June 2026.

**Why it supports.** A batch window or epoch in our venue is not an evasion
dressed as a design. The most transparent corporate bond tape in the world runs
on a 15-minute outer limit, and its regulator considered one minute and chose not
to. Whatever epoch length falls out of the per-epoch nullifier argument in
DISCLOSURE-LATTICE III.4, there is a regulator-endorsed precedent for a window of
this order in exactly our asset class.

Note the second half of the same notice cuts the other way and is worth keeping
visible: aggregated allocation reporting means one printed trade can now stand
for many managed accounts. That is aggregation reducing the resolution of the
tape, adopted in 2026, in the US, for bonds.

---

## MK-016 · the market's enforcement for conditional disclosure is a reputation score, not a proof

**Row:** order size, order price · **Mechanism:** conditional disclosure, policed
by reputation

**Source.** Turquoise service description sections 5.1, 5.3.4 and 5.3.5.

Block Discovery has our exact enforcement problem. A participant indicates a
block, is told a match exists, and must then honour the indication by submitting
a firm order. Nothing forces them to. The venue's answer is spelled out:

- A valid firm order scores 50 to 100, "depending on the size of the QBO
  relative to the original BI".
- "Failure to send a valid QBO meeting the above criteria results in a
  zero-score."
- Events combine by recency: "The most recent event has a weighting of 100, the
  next most recent 99, and so on."
- "If a user's Composite Reputational Score drops below the specified
  Reputational Score Threshold at any time, the user will be immediately
  excluded from further use of the service."
- The score is returned to the participant on FIX tag **27012**, on every OSR
  for a matched indication.

**Why it supports P3, in one comparison.** The venue's enforcement is
retrospective, probabilistic and punitive: it detects dishonesty after the fact,
scores it, and eventually ejects the participant. A proof is prospective,
deterministic and preventive: the dishonest submission cannot be constructed.
The gap between those two sentences is the product, and because the score is a
wire field with a tag number, the claim is concrete rather than rhetorical.

This is the same gap MK-004 and MK-005 found on the disclosure class, arrived at
from the opposite direction. There the flag exists and nothing proves it was
applied honestly. Here the honesty requirement is explicit and is met with a
heuristic.

---

## MK-017 · the attack the collusion check formalises is a real one, and the venue defends against it

**Row:** order size · **Mechanism:** n/a, this is an attack

**Source.** Turquoise service description sections 5.2, 5.3.1, 5.3.5 and
footnote 4.

Receipt of an OSR is one bit about a counterparty's hidden size: "at least your
own MES". One participant who can choose their own MES and repeat can vary it
across successive indications and binary-search the hidden quantity. That is not
two observers colluding, it is one observer reading the same row at many points,
which is precisely the subset MK-002 had to add to the collusion check when a
cell became a trajectory.

What the rulebook actually specifies, stated separately from our reading of it:
a Minimum Indication Value floor per instrument that an indication must meet to
be accepted at all; an eligibility floor on notifications of 25% of LIS when the
reference price waiver is allowed and 100% of LIS when it is not; automatic
expiry of the indication as soon as it is matched; reputational scoring; and
market surveillance.

**The inference, labelled as one.** The document does not describe these as
anti-probing measures. But a per-instrument floor on indication size, automatic
expiry on match, and a score that punishes not following through are together a
sensible defence against exactly the repeated-probe attack above, and there is
no other obvious reason for a floor that scales with LIS.

Either way the useful half is not in doubt: **the collusion check is modelling a
live threat, not a theoretical one**, and the one-observer-repeating case is the
one a real venue spends design effort on. That is worth saying in D9, because it
justifies running the check over trajectories rather than only over observer
subsets.

---

## MK-021 · two of four venues buy privacy the same way, and neither can prove anything afterwards

**Row:** order size, order price, pre-trade · **Mechanism:** conditional
disclosure, and permissioning, resolving to the same trust

**Source.** V3 (Turquoise sections 5.3.4, 5.3.5) set beside V4 branch B (dYdX
Chain architecture docs), plus the JELLYJELLY episode of March 2025 as the
failure mode.

Turquoise evaluates a match predicate on cleartext only it can see, and polices
honest behaviour afterwards with a recency-weighted reputation score. dYdX
validators hold the book in cleartext only they can see, and nothing afterwards
records what they held. Different sectors, different vocabulary, same structure:
**a trusted evaluator on plaintext, with no artifact that lets anyone check the
evaluation later.**

The failure mode is observable on the third venue. In March 2025 a trader opened
a roughly 4.1m short in JELLYJELLY against offsetting longs, pushed spot up more
than 400 percent in an hour, and forced the position into Hyperliquid's HLP
vault through the liquidation engine, reaching 12m to 13.5m of unrealised loss.
Validators voted within minutes to delist and settle at 0.0095, below the
prevailing price. Vault TVL fell from roughly 540m to roughly 150m over the
following weeks. The attack required the target's position and margin to be
visible in order to be sized, and **the remedy was a discretionary operator
intervention on the most transparent venue in the study.**

**Why this supports P3 rather than merely being interesting.** The pattern
across V3 and V4 is that venues do not currently have a way to be private *and*
checkable, so they pick one and backfill the other with discretion: a reputation
score, a validator set, an emergency vote. Each of those is a person deciding.
That is the gap the proof closes, and it is now visible in three venues rather
than argued from first principles.

**Source quality.** The JELLY figures are press reporting, not a chain query.
The episode is reconstructible on-chain and has not been reconstructed here, so
it is an illustration and not a measurement. **Do not put a JELLY number in the
pitch without pulling it from the ledger first.**

---

## MK-023 · given a perfectly transparent venue, the market rebuilt the dark pool on top of it

**Row:** order size, order price, pre-trade · **Mechanism:** permissioning,
retrofitted

**Source.** Flashbots documentation on Protect, which states that transactions
sent through it are hidden from the public mempool; the existence of competing
routes (MEV Blocker and others) with majority-scale validator participation.

Ethereum is the maximal element of the lattice: a settlement layer that hides
nothing from anyone at any time. If total transparency were costless, nothing
would have been built to escape it. Within a few years, private order flow
routes were built and a large share of value moved through them.

**This is the counterfactual, and it ran without us.** We do not have to argue
that participants would pay for pre-trade privacy on a transparent chain. They
already did, on the most transparent chain there is, and they paid the same
price every other venue in this study pays: the block builder now sees the
transaction before anyone else and must be trusted not to act on it, with no
artifact afterwards proving they did not.

**Source quality.** Competing figures for the private flow share came back from
secondary sources with no primary series, so no percentage is quoted. **If the
pitch needs a number, measure it or drop the sentence.** The qualitative claim
is not in doubt.

---

## MK-024 · the market proves settlement and does not prove matching, in four venues out of six

**Row:** order size, order price, and the matching decision, which is not a row
· **Mechanism:** conditional disclosure and permissioning, resolving to the same
trust

**Source.** `Polymarket/ctf-exchange`, `docs/Overview.md` and
`src/exchange/libraries/OrderStructs.sol`, read from the repository.

Verbatim: the exchange "is intended to be used in a hybrid-decentralized
exchange model wherein there is an operator that provides
matching/ordering/execution services while settlement happens on-chain,
non-custodially according to instructions in the form of signed order messages."
And: "any price improvement is captured by the taking agent."

The signatures prove both parties authorised their orders and that the swap
moved what it said. Nothing on chain shows the operator matched the best
available orders, or in the right order. The second quote prices that
discretion: the operator decides which side is the taker, and the taker captures
the price improvement. An unproven decision by a trusted party routes money on
every fill, and the settlement record cannot show whether it was fair, because
the book was never on chain.

**Four venues, four sectors, one structure.** Turquoise's operator evaluating
the match predicate (MK-013), dYdX's validators holding the book (MK-019),
Ethereum's builders seeing the transaction first (MK-023), and Polymarket's
operator assigning maker and taker. A trusted evaluator on data nobody else
sees, and no artifact afterwards. That is the market's standing answer to this
problem, and P3 is a different answer to the same one.

**Second finding, smaller and useful.** The `Order` struct carries
`address taker`, documented as "The zero address is used to indicate a public
order". That is the observer coordinate as a contract field: zero for everyone,
a named address for exactly one counterparty. MK-004 found the same dial as FIX
tags 1171 and 1172. **Two independent instantiations of the O axis, in two
technologies, neither of them ours**, and this one is in a deployed exchange
contract, which is where ours would go.

---

# H: Hedera's own record

These close D10 rows, and they are the strongest material in the folder because
they are **observed on the ledger rather than read in an announcement**. All
reproducible with [probes/hedera-register.sh](probes/hedera-register.sh), which
needs no key, no account, and no relationship with any issuer. Captured output in
[probes/hedera-register.out](probes/hedera-register.out).

## MK-008 · GOLD · the investor register of real institutional funds is public on Hedera mainnet, with exact balances
**Row:** trader identity, quantity · **Mechanism:** none
**Probed:** mainnet mirror node, unauthenticated.

`abrdn Liquidity Fund (Lux) - US Dollar Fund (L-1 Inc)`, symbol `AAULL1`, token
`0.0.9379434`. Twenty-three associated accounts are listed, nine holding a
non-zero balance, each with its exact holding:

```
  0.0.10420068     5,000,000.00 units
  0.0.10420061     1,686,994.66
  0.0.10420057     1,640,367.40
  0.0.10420056       589,827.18
  0.0.10420055       500,000.00
  0.0.10420054       500,000.00
  0.0.10420052       500,000.00
  0.0.10420058       341,914.51
  0.0.10116630     6,749,195.05   (treasury)
  ----------------------------------
  total           17,508,298.80  = total_supply, exactly
```

**The register is complete.** Holder balances reconcile to `total_supply` to the
unit, so there is no off-ledger remainder and no hidden holder. An observer who
has never spoken to the issuer knows the full participant set and every position
size in it.

**Why it matters.** F4's OUTPUT asks for "a script that reconstructs a holder
register from testnet data. The leak demonstrated, not described." This does it on
**mainnet**, against issuance by an FCA-regulated venue, for funds named after
abrdn, and the same treasury account also serves `State Street USD Treasury
Liquidity Fund (Premier)` and `BlackRock ICS US Treasury Fund (Premier Acc/Dis)`.

F4 currently argues the leak from ATS's `getKycAccountsData`. That framing is too
narrow. **The leak is not an ATS default. It is the Hedera token layer**, and it
is live under real institutional issuance today. That is a strictly stronger
motivation for the whole design and it costs nothing to claim, because anyone can
run the probe.

**Caveat kept honest:** these are share counts, not currency. A money market fund
share is conventionally near par, so the order of magnitude is safe, but no NAV
was read and none is claimed. Account ids are not attributed to named
institutions here; doing so is a separate step and is deliberately not taken.

## MK-009 · GOLD · per-account KYC status is published, revocations included
**Row:** KYC credential, compliance validity · **Mechanism:** none

`/api/v1/accounts/{id}/tokens?token.id={t}` returns `kyc_status` per account per
token. Across the funds probed, statuses are a mix of `GRANTED` and `REVOKED`.
On `0.0.9379434`, ten accounts read `GRANTED` and eleven read `REVOKED`.

This is a **public compliance register keyed by address**, at the HTS layer,
under a `kyc_key` that every probed fund sets. It is the native-Hedera analogue of
the ATS register that F4 is built around, and it has a property the ATS framing
misses: **revocation is public too.** Seeing who currently holds a credential is
one disclosure. Seeing who *lost* one is a different and more damaging one, and
nothing in our matrix has a row for it yet.

**Open, and it is a real question for D9:** F4's nine rows have no entry for
credential *history*. `KYC credential` covers the current bit. The transition
`GRANTED -> REVOKED`, publicly timestamped, is a tenth item or it is an explicit
part of row 2. Decide it in D9 rather than discovering it in the demo.

## MK-010 · onboarding cohort and sequence are recoverable from account ids alone
**Row:** trader identity · **Mechanism:** none

Six of the holders were created inside one nine-minute window:

```
  0.0.10420052   2026-04-04T16:09:41Z
  0.0.10420055   2026-04-04T16:10:50Z
  0.0.10420057   2026-04-04T16:11:42Z
  0.0.10420061   2026-04-04T16:13:09Z
  0.0.10420068   2026-04-04T16:18:09Z
```

Hedera account ids are monotonically assigned, so ordinality alone is a
timestamp proxy and the creation timestamp confirms it exactly. That gives an
observer the onboarding cohort, its order, and its date, for free.

**This is mu2.4 observed in production.** P2's micro-problem asks whether
credential issuance time correlates against first trade time. Here the issuance
time is not merely correlatable, it is *published to the second*, and cohort
membership is inferable from the account id without querying anything.

**Design consequence, and it survives our own design.** Our registration
transaction (F5b's two-transaction structure) creates exactly this artefact: a
funded address, created at a known instant, in an id range shared with everyone
else onboarded that afternoon. A per-epoch nullifier breaks the join *across
epochs*; it does nothing about a cohort visible at account creation. Either
registration addresses are pre-existing and unremarkable, or the epoch boundary
has to be coarse enough that a cohort is not a fingerprint. This belongs in
`SECURITY-MODEL.md` next to the nullifier argument.

## MK-011 · one treasury account serves competing managers' funds, publicly
**Row:** asset, trader identity · **Mechanism:** none

Account `0.0.10116630` is the treasury for the abrdn Sterling, Euro and US Dollar
liquidity funds, the State Street USD Treasury Liquidity Fund, and the BlackRock
ICS US Treasury funds. The commercial relationship between the venue and five
fund ranges across three competing managers is readable from the token index.

Low severity on its own. Logged because it is the clean example of a leak that no
row in F4 covers: nothing about a *counterparty relationship* is a data item in
the inventory, yet F4's own prose lists "who their counterparties are" among the
things an observer reconstructs. Either the inventory is missing a row or the
prose is overclaiming. D9 has to pick one.

## MK-012 · the documentation is silent on every one of the above
**Row:** meta · **Mechanism:** n/a

Three sources on the July 2025 Archax / Lloyds / abrdn trade: the Hedera case
study, the Lloyds Banking Group press release, and Ledger Insights. Between them
they name **no token id, no transaction id, no network (mainnet or testnet), and
make no statement about what is visible on the ledger versus held off-chain.**
Archax's "Nest" permissioned collateral network is mentioned without a
description of what permissioning covers.

F3b's instruction for a permissioned venue was to "record where the public
account goes quiet, because that silence is the permissioning mechanism." The
finding here is sharper than that. The silence is not concealment: the data is
wide open, as MK-008 shows. **The disclosure decisions were never documented
because they were never made.** What is public is whatever the token layer
defaults to.

That is the D10 paragraph the plan asks for, and it is the answer to old section
1's question about what Hedera's institutional record demands that the brief did
not ask for.

---

# PARKED

Real, sourced, and not ours to chase on a twelve-day clock. One line each.

- **MiFIR Article 8a pre-trade transparency for bonds.** The pre-trade half of the
  regime, restructured in the same review. We use the post-trade half. If a judge
  asks about pre-trade waivers, read this before answering.
- **The MiFIR consolidated tape.** Delegated regulation on reasonable commercial
  basis, adopted alongside RTS 2. Changes who pays for the tape, not who sees it.
- **HQLAx.** The collateral leg of Kinexys intraday repo. Relevant to F1 repo
  mechanics, not to a disclosure cell.
- **Whether the volume cap actually protects price formation.** The empirical
  market-structure literature on dark trading share and quote quality. MK-015
  needs only that the cap exists and that D9 cannot represent it, and both are
  settled from the text. Whether the cap is good policy is a research programme
  and is not ours.
- **EU sovereign supplementary deferrals by member state.** Each member state
  decides for its own issued debt; ESMA decides for the rest. A per-jurisdiction
  patchwork, and a good illustration that the policy is configuration, but it
  fills no cell we do not already have from MK-001.

---

## MK-025 · the consensus node that received a transaction is public, and this venue pins it to a competitor's node
**Row:** none. A new axis · **Mechanism:** none

**Source.** `mainnet-public.mirrornode.hedera.com/api/v1/transactions`, field
`node`, in [probes/derived-rows.sh](probes/derived-rows.sh).

Every mirror node transaction record carries the account id of the consensus node
that received it. `[MEASURED]`: **all 190 token transactions on the
shared treasury `0.0.10116630` were submitted to node `0.0.29`**, and 109 of 111
on counterparty `0.0.10420070`. Node `0.0.29` is operated by **Aberdeen
Investments**, one of the five managers whose funds sit on that treasury.

**Control.** Across 100 random recent mainnet transactions, traffic spreads over
28 distinct nodes with the busiest at 7.0 percent. So 100 percent on one node is
a deliberate pin, not an SDK default.

**Three things follow, and none is an allegation.**

1. Every transaction on this treasury, across five competing managers, is handed
   first to a node operated by one of them, during the exclusive pre-gossip
   window [C-05](../mesh/consensus-visibility.md) identified.
2. **Node pinning is existing production practice**, which de-risks D-15
   considerably. We are not proposing a novel transport trick.
3. The pin is **itself disclosed, permanently**. Choosing your first observer is
   possible on Hedera and it is not confidential. What production is missing is
   not the pin, it is treating the pin as a disclosure with a policy.

Filed as residual X2 in [F4-ROW-SCOPE.md](F4-ROW-SCOPE.md): a property of the
transport, so no cell of a per-trader matrix can hold it.

---

## MK-026 · a tokenised US Treasury bill is already live on Hedera mainnet, and it is repo collateral
**Row:** asset · **Mechanism:** none

Token `0.0.10633046`, name `US912797UJ40`, on the same shared treasury as the
money market funds. `[MEASURED]` **216,014,405 units issued in a single
transfer**, publicly, with sender, recipient, amount and timestamp all readable
without an account.

MK-008 anchored the pitch on tokenised money market funds. A tokenised T-bill is
a **better anchor**, because it is collateral rather than a fund wrapper, which
is one step from the repo use case this project exists to serve rather than two.
The disclosure position is identical and equally bad: the entire issuance is
public at full granularity to all observers immediately.

Also corrects MK-011's count. The treasury serves **five** competing managers,
not three: abrdn, BlackRock, State Street, Fidelity and LGIM.
