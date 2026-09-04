# V6. Polymarket: the aggregate is the product

**Status: filled.** The row closes the taxonomy question the plan set for it,
and it produces the sharpest one-line statement of our gap found anywhere in the
study.

**Primary sources.** `Polymarket/ctf-exchange`, read from the repository:
`src/exchange/libraries/OrderStructs.sol` and `docs/Overview.md`. Revenue
measured in [probes/hyperliquid-positions.sh](../probes/hyperliquid-positions.sh)
section 4, which reads the whole fees table rather than one protocol.

---

## 1. The observer axis, in Solidity

The `Order` struct is signed EIP-712 typed data. Twelve fields are in the type
hash. One of them is not about the trade at all:

```solidity
/// @notice Address of the order taker. The zero address is used to indicate a public order
address taker;
```

That is the observer coordinate, as a contract field. Zero address means the
order is offered to everyone; a named address means it is offered to exactly one
counterparty. It is the same dial MK-004 found as FIX tags 1171 and 1172, and it
is more useful to us because it is in a deployed exchange contract rather than a
message spec. **Two independent instantiations of the O axis, in two
technologies, neither of them ours.**

The rest of the struct is the trade: `salt`, `maker`, `signer`, `tokenId`,
`makerAmount`, `takerAmount`, `expiration`, `nonce`, `feeRateBps`, `side`,
`signatureType`, plus the signature. Everything a matching engine needs, signed
by the trader, and settled by a contract that never saw the book.

---

## 2. What the venue proves, and the sentence it hands us

From `docs/Overview.md`, verbatim:

> "It is intended to be used in a hybrid-decentralized exchange model wherein
> there is an operator that provides matching/ordering/execution services while
> settlement happens on-chain, non-custodially according to instructions in the
> form of signed order messages."

And, two sentences later:

> "When orders are matched, one side is considered the maker and the other side
> is considered the taker. The relationship is always either one to one or many
> to one (maker to taker) and any price improvement is captured by the taking
> agent."

Put those together and the gap states itself:

> **Polymarket proves settlement and does not prove matching.**

The signatures prove that both parties authorised their own orders and that the
swap moved the tokens it said it would. Nothing on chain shows that the operator
matched the best available orders, or in the right order. And the second quote
gives that discretion a price: the operator decides which side is the taker, and
the taker captures the price improvement. So an unproven decision by a trusted
party routes money between two counterparties on every fill, and the settlement
record cannot show whether the decision was fair, because the book it was made
against was never on chain.

**Fourth instance of the pattern.** Turquoise's operator evaluating the match
predicate, dYdX's validators holding the book, Ethereum's builders seeing the
transaction first, and now Polymarket's operator assigning maker and taker. Four
venues, four sectors, one structure: a trusted evaluator on data nobody else
sees, and no artifact afterwards. This is no longer a pattern we noticed. It is
the market's standing answer to the problem, and P3 is a different answer to the
same problem.

---

## 3. The nine rows

| F4 row | G | O | T | Mechanism |
|---|---|---|---|---|
| Trader identity | pseudonymous address | all, post-settlement | t₀ | none |
| KYC credential | jurisdictional gating, off chain | operator | pre-relationship | permissioning |
| Order size | `makerAmount` / `takerAmount`, full | operator pre-match; all post-settlement | at order, then t₀ | conditional, weakly |
| Order price | implied by the amount pair, full | operator pre-match; all post-settlement | at order, then t₀ | conditional, weakly |
| Execution price | full | all | t₀ | **none, by design** |
| Quantity | full | all | t₀ | none |
| Asset | full (`tokenId`) | all | t₀ | none |
| Compliance validity | absent on chain | | | none |
| Settlement validity | full, atomic swap, signature-checked | all | t₀ | none |

The `taker` field can move rows 3 and 4 to a single named counterparty, which is
why they read "conditional, weakly": the mechanism exists in the struct, and the
venue's normal operation is public orders.

---

## 4. Why execution price cannot be hidden here, and what that settles

Polymarket exists to produce a number. The price of an outcome token *is* the
forecast, and the forecast is the product the venue sells. A prediction market
that deferred its own execution price would have deferred the only thing anyone
came for.

This is the cleanest available confirmation that D9's treatment of execution
price is right in kind. Across the six venues, execution price is the one row
that is public everywhere: immediately here and on Hyperliquid and Uniswap, and
after a stated deferral in the bond market and on the dark venues. **No venue
studied hides its execution price permanently from everyone.** D9 places it at
`(full, all, t₀+δ)`; the venue evidence says the coordinate is right and only δ
is in dispute, running from zero, where price is the product, to end of day for
sovereign volume, per MK-001.

That is the "hidden by none" half of the paragraph F3b's OUTPUT asks for, and it
is now settled by six rows rather than assumed.

---

## 5. What an observer reconstructs about one trader

The plan set this row the question specifically. The answer is: everything,
after settlement, because positions are ERC-1155 balances and fills are
contract events on a public chain. Address, position per outcome token, entry
amounts, and the full history are recoverable by anyone willing to index the
chain, and Polymarket publishes convenience APIs over exactly that.

This is F4's activity leak in the wild, with two differences from the Hedera
finding in MK-008. The identity is pseudonymous rather than a named institution,
and the venue is not pretending otherwise. But the reconstruction is the same
operation, and the same rebuttal applies to both: pseudonymity is not a
disclosure class, it is an unproven assumption that nobody will link the
address.

Commercially the venue is not marginal. On the same measurement that ranked
Hyperliquid third, **Polymarket International is ninth at 15.7 million USD of
30-day revenue**, ahead of every DEX in the table.

---

## 6. The two questions

**What is hidden, from whom, until when, by which mechanism?** Before matching,
the resting order is visible to the operator and, in the `taker`-addressed case,
to one named counterparty. After settlement, nothing is hidden from anyone. The
mechanism is a weak conditional disclosure in the struct, and permissioning at
the operator.

**What does the venue prove, and who is trusted?** It proves settlement:
authorisation, atomicity, and the token movement. It does not prove matching,
and the operator who is trusted for matching also assigns the side that captures
price improvement.

---

## Honest limits

- Contracts are read from the `main` branch of the public repository. A v2
  exchange exists and was not read; if the pitch cites Polymarket mechanics,
  confirm which version is live first.
- No probe was run against a live position. The reconstruction claim in §5
  follows from ERC-1155 balances and public events, which is sound, but it is
  reasoned rather than demonstrated, unlike MK-008 which was measured.
- Resolution and the oracle path (UMA) are not read. They matter for settlement
  risk and not for a disclosure cell, so they are parked.
