# V4. Perpetual DEXes: where the book lives before consensus

**Status: filled, and branched.** This row is the empirical companion to μ1.4,
and it turned into the architecture decision rather than a comparison.

**Primary sources.** Hyperliquid's public API, queried directly:
[probes/hyperliquid-positions.sh](../probes/hyperliquid-positions.sh), output in
[hyperliquid-positions.out](../probes/hyperliquid-positions.out). dYdX Chain
architecture documentation (`docs.dydx.xyz/concepts/architecture/overview` and
the limit orderbook page). Revenue measured from the DefiLlama fees API inside
the same probe, not quoted from an article.

---

## The branch point

The plan framed this as two venues. It is better read as one question with three
answers, because the third is ours and it only becomes visible once the first
two are laid side by side.

> **Can a replicated order book be replicated without being disclosed?**

Every design here is an answer to that, and each answer pays somewhere
different.

| | **A. Hyperliquid** | **B. dYdX v4** | **C. ours** |
|---|---|---|---|
| Where the book lives | HyperCore, in consensus | validator memory, off consensus | consensus, as a commitment |
| What is replicated | the plaintext book | nothing; orders are gossiped | a commitment plus a validity proof |
| Who sees an unfilled order | everyone, immediately | every validator | nobody |
| Who sees a fill | everyone | everyone | per the disclosure matrix |
| Is disclosure a policy? | **no, it is a consequence** | partly | **yes, it is chosen per row** |
| Auditable afterwards | fully | **not at all** | by proof |
| Price paid | total information leak | trust in the validator set, unverifiable | proving cost, circuit risk |

**The observation that makes the row worth doing.** In branch A, disclosure is
not a decision anyone made. Byzantine fault tolerance over an order book
requires every validator to hold the same book, and a validator holding the book
*is* the disclosure. The public API does not leak anything consensus had not
already required. You cannot patch privacy into branch A, because the leak is
the replication.

Branch B does not solve that. It relocates it. The book still sits in the clear
in every validator's memory; it just never reaches consensus, so the public sees
only fills. That converts a public leak into a leak to a fixed set of operators,
which is **permissioning**, mechanism 1 from F3b's table, applied to a set that
markets itself as decentralised. And it costs something A does not pay: because
resting orders are never committed, there is no record against which anyone can
later check that the matching was honest.

That is the same trade V3 found at Turquoise, under different branding. A
trusted party evaluates on cleartext, and nothing afterwards proves they did it
right. **Two of the four venues studied so far land in the same place**, which
is a stronger pattern than either row on its own.

---

## Branch A, measured

Everything below was read from the live API with no key, no account and no
signature.

| F4 row | G | O | T | Mechanism |
|---|---|---|---|---|
| Trader identity | pseudonymous address, persistent and ranked | all | immediate | none |
| KYC credential | absent | | | none |
| Order size | full, per resting level | all | on placement | none |
| Order price | full, 20 levels each side | all | on placement | none |
| Execution price | full | all | immediate | none |
| Quantity | full | all | immediate | none |
| Asset | full | all | immediate | none |
| Compliance validity | absent | | | none |
| Settlement validity | full, one-block finality | all | immediate | none |

Nine rows, no waiver, no deferral, no mechanism anywhere. This is the maximal
element of the lattice, running in production and earning more than any other
protocol in crypto.

The register: `stats-data.hyperliquid.xyz/Mainnet/leaderboard` returns
**44,627 accounts**, each with an address, an account value, and day, week,
month and all-time PnL and volume. Feed any address to
`api.hyperliquid.xyz/info` and you get its live book:

```
0x5b5d51...c060   accountValue=81,913,175
   BTC  size=-1707.27249   entry=70551.3   LIQUIDATION=124904.26132943   5x cross
   ETH  size=-55114.1154   entry=2124.08   LIQUIDATION=3888.6625421988   5x cross
   SOL  size=-788679.41    entry=93.6161   LIQUIDATION=201.0069217427   10x cross
```

An 81 million dollar account, short 1,707 BTC, and **the exact price at which it
will be forcibly closed**, to eleven significant figures, free, to anyone, in
real time. Everything an adversary needs to size a squeeze is a field in a
public response body.

### The field that is not in F4

`liquidationPx` is not one of the nine rows. It is a function of position size,
margin, leverage and the mark price, and it is by some distance the most
dangerous number on the account. This is the **fourth independent sighting** of
the defect MK-014 named: credential history (MK-009), counterparty relationships
(MK-011), the match predicate (MK-013), and now the liquidation price. F4
enumerates primitive fields; the leaks that get people hurt are derived facts
over them. After four sightings that is no longer an observation, it is a
confirmed structural gap, and D9 has to close it.

---

## The commercial collision, which is the real content of this row

Measured in the probe, from DefiLlama's fees API, 30-day revenue retained:

| Rank | Protocol | 30d revenue USD | Category |
|---|---|---|---|
| 1 | Tether | 478,312,617 | Stablecoin Issuer |
| 2 | Circle USDC | 191,083,139 | Stablecoin Issuer |
| **3** | **Hyperliquid Perps** | **49,145,903** | **Derivatives** |
| 4 | Canton | 48,821,841 | Chain |
| 5 | pump.fun | 35,889,956 | Launchpad |
| 9 | Polymarket International | 15,727,271 | Prediction Market |

The two above it are issuers earning on reserves, not venues. **Among trading
venues Hyperliquid is first, by roughly three times**, with 1.18 billion dollars
of all-time revenue. It also outearns every chain in the table, Tron included.

That is the hardest fact this project has to face, and it should be stated in
its strongest form rather than managed: *the most commercially successful
trading venue in crypto discloses everything, and a privacy layer is not what
made anyone rich.*

**The resolution, and it is a real one, not a save.** Transparency is affordable
exactly where the leaked information is cheap. A Hyperliquid position is in a
liquid, continuously traded, globally fungible perp; being read costs basis
points on the way in. A block in a single illiquid bond is the opposite case,
which is why the LIS waiver exists at all, and why every venue in V1, V2 and V3
that serves institutions hides. The segments are different, and the evidence for
that is negative and checkable: **there is no institutional bond or repo
activity on Hyperliquid.** The absence of the customer we are building for, on
the most successful venue in the space, is the datum.

Nor is the leak free even here. In March 2025 a trader opened a roughly 4.1
million dollar short in JELLYJELLY against offsetting longs, then pushed the
spot price up more than 400 percent in an hour, forcing the position into
Hyperliquid's own HLP vault through the liquidation engine, where it ran to
between 12 and 13.5 million dollars of unrealised loss. Validators voted within
minutes to delist the market and settle it at 0.0095, below the prevailing
price, and vault TVL fell from about 540 million to about 150 million over the
following weeks. The attack needed the target's position and margin to be
visible in order to be sized, and the venue's remedy was a discretionary
operator intervention.

So the honest scoreboard on branch A is: transparency is commercially proven,
demonstrably not free, and its failure mode is resolved by exactly the trusted
operator the design was supposed to eliminate.

---

## Branch C, and what this row decides for Hedera

μ1.4 asks whether Hedera consensus node operators see transactions before
consensus. The three branches above turn that from a curiosity into a
constraint, because the answer places us in a branch whether we choose one or
not.

If consensus nodes see plaintext transactions before ordering, then an order
book hosted on Hedera **is branch B by default**, with the Governing Council as
the validator set. That is not an abstract concern here. The Council is 31
members and includes financial institutions: Standard Bank, DBS and Shinhan Bank
are all members. In an institutional repo market, order flow visible to
consensus node operators is order flow visible to organisations that may be
counterparties, affiliates of counterparties, or competitors of them.

Nothing in that sentence is an accusation, and it does not need to be. Branch B
is unacceptable for this use case on structure alone, because the property a
regulated participant needs is that the question does not arise. That is the
argument for branch C, stated without reference to anyone's conduct, and it is
the strongest version available.

**So the design conclusion is forced, not chosen:** the book must reach
consensus as a commitment, because branch A leaks to everyone, branch B leaks to
the Council, and only branch C makes disclosure a policy again rather than a
consequence of replication.

---

## The two questions

**What is hidden, from whom, until when, by which mechanism?** Branch A hides
nothing, from nobody, ever, by no mechanism. Branch B hides resting orders from
the public and from nobody else, by permissioning, until a fill.

**What does the venue prove, and who is trusted?** Branch A proves everything
and trusts nobody, which is precisely why it can hide nothing: its auditability
and its leak are the same property. Branch B proves only that committed fills
were applied, and trusts every validator with the order flow, with no record
afterwards to check the matching against. Neither venue occupies the position we
are claiming, and A shows why it is not free to claim it.

---

## Honest limits

- The dYdX branch is read from documentation, not probed. It deserves the same
  treatment branch A got, and until it gets it the claim that resting orders
  never reach consensus rests on the docs saying so.
- The JELLY figures come from press reporting, not from a chain query. The
  incident is reconstructible on-chain and is not, so it is cited as an
  illustration rather than as a measurement. **Do not put a JELLY number in the
  pitch without pulling it from the ledger first.**
- The claim that Hyperliquid carries no institutional bond or repo activity is
  an argument from absence. It is true of the instrument list, which is perps,
  and it should be stated as that rather than as a survey of who trades there.
- Revenue is a 30-day window measured on one day. It moves. The probe reruns.
- μ1.4 itself is still open. This row states what follows from each answer; it
  does not answer it.
