# The six venues, compared on what they hide

This is `OUTPUT F3b`. Six venues against F4's nine rows, plus the paragraph the
plan asks for: which cells every venue hides, which none hide, and where our
design sits that none of the six do.

Each row below is filled from its own dossier in [venues/](venues/), each of
which cites a primary source and answers the trust question. Nothing here is new
evidence; it is the six rows read across instead of down.

---

## 1. The matrix, pre-trade

Pre-trade, because post-trade nearly everything converges to public and the
interesting differences vanish. `hidden` means withheld from the public.
`selective` means disclosed to a chosen few. `public` means anyone can read it.

| F4 row | V1 OTC bonds | V2 Kinexys | V3 dark pools | V4 Hyperliquid | V5 Uniswap | V6 Polymarket |
|---|---|---|---|---|---|---|
| Trader identity | selective | hidden | hidden | pseudonymous | pseudonymous | pseudonymous |
| KYC credential | hidden | hidden | hidden | absent | absent | hidden |
| Order size | selective | hidden | hidden | **public** | **public at t₀−δ** | hidden |
| Order price | selective | hidden | hidden | **public** | **public at t₀−δ** | hidden |
| Execution price | public +δ | hidden | public +δ | **public** | **public** | **public** |
| Quantity | public +δ, capped | hidden | public +δ | **public** | **public** | **public** |
| Asset | selective | hidden | public +δ | **public** | **public** | **public** |
| Compliance validity | hidden | hidden | hidden | absent | absent | hidden |
| Settlement validity | hidden (CCP) | hidden | hidden (CCP) | **public** | **public** | **public** |

V4 is shown as Hyperliquid. Its dYdX branch differs in one cell: order size and
price are hidden from the public and visible to every validator, which is
selective disclosure to an operator set. See
[04-perp-dexes.md](venues/04-perp-dexes.md).

---

## 2. Hidden by every venue

Three rows, and they are the same three:

- **Trader identity.** No venue in the study publishes who is trading. The
  traditional three hide it behind a counterparty or an operator; the crypto
  three replace it with an address. Different mechanisms, same cell.
- **KYC credential.** Where it exists it is never published. It sits with a
  dealer, an operator, or a jurisdiction gate.
- **Compliance validity.** Nowhere is it publicly checkable. It is either absent
  entirely, or it is asserted privately by whoever is trusted.

## 3. Hidden by no venue

- **Execution price**, permanently, from everyone. Every venue publishes it:
  immediately where price is the product (V4, V5, V6), after a stated deferral
  where size moves the market (V1, V3). This settles D9's most contested cell.
  The coordinate `(full, all, t₀+δ)` is right in kind across six independent
  venues, and only δ is in dispute, running from zero to end of day per MK-001.
- **Asset** and **quantity**, eventually. Both are deferred or capped somewhere,
  and permanently withheld nowhere.

## 4. Where we sit that none of the six do

Rows 2 and 3 are almost the same list read twice, which is the finding.

**Every venue hides identity and compliance together, and every venue publishes
price and quantity together.** Nobody separates them. The reason is that all six
have exactly one tool for hiding a thing, which is to make it invisible to
whoever should not have it, and an invisible compliance assertion is worth
nothing to anyone except the party trusted to have checked it.

Our position is the one cell six venues leave empty: **hide the identity and
publish a proof of the compliance assertion attached to it.** Not the credential,
not the identity, not the counterparty, but a publicly checkable statement that
the trade satisfied the rules that applied to it. No venue studied does that,
because none of them can: it requires a mechanism none of the four hypothesised
had, and the fifth one they do have (conditional disclosure, MK-013) still needs
a trusted evaluator.

---

## 5. The 2x2, which is the pitch slide

Plot the six on two axes: is the pre-trade information hidden, and is what the
venue asserts about the trade checkable by someone who was not there.

| | **not checkable** | **checkable** |
|---|---|---|
| **public** | | Hyperliquid, Uniswap (base), Polymarket settlement |
| **hidden** | OTC bonds, Kinexys, dark pools, dYdX validator set, private order flow, Polymarket matching | **empty** |

The bottom-right cell is empty, and it is not empty because nobody wanted it.
Four of the six venues put real engineering into the bottom-left: Turquoise's
reputational score, dYdX's validator set, Ethereum's builder market, Polymarket's
operator. Each of those is a person or a set of people deciding, backfilled with
whatever discretion was available.

**The market has one standing answer to "private and checkable", and the answer
is a trusted evaluator.** P3 is a different answer to the same question, and the
six rows establish the question is real rather than invented.

---

## 6. What the study changed on our side

Not a summary. Only the things that alter what gets built.

1. **The mechanism taxonomy was wrong.** Four became five: conditional
   disclosure runs in production at Turquoise and in MiFIR's iceberg waiver
   (MK-013, `AB-033`). It is the mechanism nearest our own, and the correct
   reading strengthens P3 rather than weakening it.
2. **F4's nine rows are the wrong nine.** Four sightings of the same gap:
   credential history (MK-009), counterparty relationships (MK-011), the match
   predicate (MK-013), the liquidation price (MK-020). The sensitive facts are
   *derived* from the primitive fields, and the matrix has no row for them.
3. **The lattice is not a clean product.** `O` can be computed from the data
   (MK-014), a cell has to be a trajectory rather than a point (MK-002), and δ
   is per-row rather than per-venue (MK-001).
4. **The time axis needs a point before zero** (MK-022), which is where the
   largest measured losses occur, and which makes μ1.4 a deciding question
   rather than a cheap aside.
5. **There is a second-order constraint no cell can express** (MK-015): MiFIR's
   volume cap meters how much of a market may hide at all.
6. **The architecture is forced, not chosen** (`AB-034`). A book in consensus
   cannot express a disclosure policy; a book in validator memory can only
   express one about the public. Only a committed hidden book restores per-row
   choice, and on Hedera the default is the middle case with the Council as the
   operator set.

---

## 7. What is still open

- μ1.4, now the deciding probe rather than a cheap aside.
- Which derived rows are in scope for F4. Four are known; there is no principled
  stopping rule yet, and inventing one is D9's job.
- The consolidated MiFIR Article 5 text, for MK-015's number.
- The dYdX branch, read from documentation and not probed.
- H3, the remaining Hedera institutional deployments.
