# V5. Uniswap: total transparency, priced, and the counterfactual that ran

**Status: filled.** Read as counterexample and as settlement primitive, per the
plan, not as an architecture we might pick.

**Primary sources.** Uniswap v4 developer documentation on flash accounting and
the unlock callback. Qin, Zhou and Gervais, *Quantifying Blockchain Extractable
Value: How dark is the forest?*, IEEE Symposium on Security and Privacy 2022
(arXiv 2101.05511), for the one measurement whose method we can restate.

---

## 1. The nine rows, and a coordinate D9 does not have

| F4 row | G | O | T | Mechanism |
|---|---|---|---|---|
| Trader identity | pseudonymous address, persistent | all | **t₀ − δ** | none |
| KYC credential | absent | | | none |
| Order size | full | all | **t₀ − δ** | none |
| Order price | full (limit implied by slippage bound) | all | **t₀ − δ** | none |
| Execution price | full | all | t₀ | none |
| Quantity | full | all | t₀ | none |
| Asset | full | all | **t₀ − δ** | none |
| Compliance validity | absent | | | none |
| Settlement validity | full, atomic | all | t₀ | none |

**The finding is in the T column.** Every venue studied so far discloses at t₀ or
later: at the trade, at the tape, after a deferral. A swap broadcast to the
public mempool is disclosed **before it happens**, to anyone running a node, and
it may never happen at all. Order size, price bound, asset and sender are all
public while the trade is still only an intention.

D9's rows all sit at t₀ or t₀+δ. The time axis needs a point strictly before
zero, and it is the only point on the axis where disclosure and execution can be
reordered by the observer. That is not a curiosity: **t₀ − δ is the coordinate
sandwiching lives at**, and it is the same coordinate μ1.4 is asking about for
Hedera. See `AB-034`.

---

## 2. The measurement, with its method

Qin, Zhou and Gervais define a transaction-ordering taxonomy, apply it to
historical Ethereum state, and attribute value to sandwich attacks, liquidations
and DEX arbitrage. Over 32 months they attribute **540.54 million USD** of
extracted value across 11,289 addresses, over 49,691 assets and 60,830 on-chain
markets. The largest single instance is 4.1 million USD, which they note is
616.6 times the block reward of the block it sat in.

The method restates in one sentence, which is why this citation and not a
dashboard: *classify each transaction by its position relative to the
transactions it profits from, then price the difference.* It is reproducible,
peer-reviewed, and it measures the cost of the t₀ − δ disclosure specifically
rather than "MEV" in general.

**Read carefully, this is P1's claim priced.** It is not the price of publishing
a trade. It is the price of publishing an *intention* to trade, in a venue where
anyone can act on it first.

---

## 3. The counterfactual, which is the strongest thing in this row

Ethereum is the maximal element of the disclosure lattice: a settlement layer
that hides nothing from anyone at any time. If total transparency were costless,
nothing would have been built to escape it.

Something was built to escape it. Private order flow routes now take user
transactions directly to block builders instead of the public mempool.
Flashbots' own documentation states that transactions sent through Protect are
hidden from the public mempool. Multiple such routes compete (Flashbots Protect,
MEV Blocker, and others), and validator participation in them is
majority-scale.

**Given a perfectly transparent venue, the market rebuilt the dark pool on top of
it, within a few years, and moved a large share of value through it.** The user
gains privacy at t₀ − δ. The price is that the block builder now sees the
transaction before anyone else and must be trusted not to act on it, with no
artifact afterwards proving they did not.

That is the same trade V3 found at Turquoise and V4 found in dYdX's validator
set. **Third instance, third sector, identical structure:** a trusted evaluator
sees plaintext early, and nothing afterwards is checkable. See MK-021.

---

## 4. Atomic settlement, the primitive F6b needs

Uniswap v4 is a singleton `PoolManager`. No interaction is possible until a
caller invokes `unlock`, which hands control back to the caller's
`unlockCallback`. Inside that callback, operations do not move tokens; they
accumulate **deltas**. Negative deltas are paid by transferring and calling
`settle`, positive deltas are claimed with `take`, and the unlock cannot
complete unless every delta has netted to zero.

Stated as an invariant, which is the form F6b wants it in:

> A state transition is valid only if the sum of all balance deltas it produces
> is zero at the moment the lock is released.

That is exactly the shape of an invariant a circuit proves. It is enforced here
by a contract that can read every value; we would enforce the same statement
over values the verifier cannot read. The primitive transfers directly, and
noting that it is *already* expressed as a net-to-zero check on deltas rather
than as a sequence of transfers is worth carrying into F6b, because the
delta form is the one that survives being hidden.

---

## 5. The two questions

**What is hidden, from whom, until when, by which mechanism?** In the base case,
nothing, from nobody, ever. In the private order flow case, the intention is
hidden from the public mempool and disclosed instead to a block builder, by
permissioning, until inclusion.

**What does the venue prove, and who is trusted?** The AMM proves everything and
trusts nobody: the pool's arithmetic is public and the net-zero invariant is
enforced on chain. That is a genuinely strong position, and it is available only
because nothing is hidden. Once privacy is added at t₀ − δ by routing around the
mempool, the proof does not extend to cover the new trusted party, and no
artifact records what the builder saw.

---

## Honest limits

- The private order flow share is stated qualitatively. The searches returned
  competing figures from secondary sources and no primary series, so no
  percentage is quoted here. **If the pitch needs a number, measure it or drop
  the sentence.** The qualitative claim, that such routes exist, carry
  significant flow and are the market's own answer to mempool transparency, is
  not in doubt.
- The BEV measurement covers Dec 2018 to Aug 2021. It is the best method-stated
  figure available, and it is old. It is used here for the mechanism and the
  order of magnitude, not as a current number.
- v4 mechanics are read from documentation, not from the deployed bytecode.
- MEV mechanism design, builder market structure and order flow auctions are all
  parked, per the plan's scope discipline.
