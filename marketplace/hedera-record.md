# D10. Hedera's institutional record, read off the ledger

**Method note, and it is the whole reason this file looks different from the
venue dossiers.** D10 asks: for each real institutional deployment on Hedera,
what did the participants choose to disclose, and what did they choose to
withhold? The plan assumed that question is answered by reading announcements.

It is not. It is answered by querying the ledger, because
[MK-012](EVIDENCE.md#mk-012--the-documentation-is-silent-on-every-one-of-the-above)
found that the announcements say nothing about disclosure at all: no token id, no
transaction id, no network named, no visibility statement. So every row below is
either **observed** (a probe was run, the row is a fact) or **open** (the probe is
named but not yet run). No row is filled from a press release.

Probe: [probes/hedera-register.sh](probes/hedera-register.sh). Output:
[probes/hedera-register.out](probes/hedera-register.out). Neither needs a Hedera
account, a key, or any relationship with an issuer.

---

## The table

| Deployment | What was public | What was withheld | Which D9 cell it evidences | State |
|---|---|---|---|---|
| **Archax tokenised MMFs** (abrdn Sterling / Euro / USD liquidity funds, State Street USD Treasury Liquidity, BlackRock ICS US Treasury), mainnet | The complete holder register with exact per-account balances, reconciling to `total_supply` to the unit. Per-account `kyc_status`, `GRANTED` and `REVOKED`. Account creation timestamps and ordinal ids. Token metadata: treasury, supply, which keys are set. The treasury account shared across three competing managers. | Nothing that the token layer publishes by default. Legal identity behind each account id is not on the ledger, and no attempt to attribute it was made here. | **Trader identity**, **quantity**, **KYC credential**, **compliance validity**. All four evidenced as `(full, all, immediate)` in production, which is the coordinate our design exists to move. | **observed** 2026-08-31 |
| **Archax / Lloyds / abrdn tokenised collateral FX trade**, 14 July 2025, "UK-first use of digital assets", tokenised abrdn MMF units and tokenised UK gilts as collateral | Unknown from documentation. The trade is described; nothing about its ledger footprint is. Archax's "Nest" permissioned collateral network is named without a description of what the permissioning covers. | Unknown. | The row that would matter most, because it is our exact use case: tokenised gilts as collateral, UK, institutional, on Hedera. | **open**, probe below |
| **ATS shipped defaults** | `getKycAccountsData` publishes the investor register; transfer log is public; five external seams reached at transfer, two fail open. | `internalKycActivated = false` at issuance suppresses the register entirely (`AB-001`). | The baseline our overlay is a diff against. | **done**, see [mesh/transfer-path.md](../mesh/transfer-path.md) |
| **Woolard report / 54-firm taskforce** | Public commitment; collateral mobility across platforms named as the problem. | n/a | No disclosure cell. Pitch context, and the roadmap gap (DIGIT targeted early 2027 on HSBC Orion, not Hedera). | open, low value for D9 |
| **Other Council member deployments** | | | | open |

---

## The open probe, and why it is worth an hour

**Question.** Is the July 2025 collateral movement visible on mainnet, and if so,
at what granularity: the tokenised gilt, the MMF units, the direction, the size,
the timing?

**How to run it.** The token index is searchable by name substring without
authentication, which is how MK-008's tokens were found. `gilt` currently returns
nothing, so either the gilt token is named differently, was issued on a private
network, or has been deleted. Three next steps, in order of cost:

1. Enumerate tokens whose treasury is `0.0.10116630` or `0.0.6827532`, the two
   Archax-operated accounts already identified. Deleted tokens still appear in
   mirror node history.
2. Pull the transaction history of those accounts around 2025-07-14 and look for
   `CRYPTOTRANSFER` with token transfers on that date.
3. If the trade is visible, record exactly what an observer learns: which
   direction, what size, against which collateral, at what time.

**Why it is worth doing and why it is not urgent.** Either answer is useful. If
the trade is visible, the pitch has a named, dated, UK-first institutional
transaction whose economics are legible to anyone with curl, which is F4's
activity leak on real flow rather than on a demo. If it is not visible, that is
also worth knowing, because it means Archax did something deliberate and we
should understand what.

It is not urgent because MK-008 already establishes the register leak on mainnet
under live issuance, which is the load-bearing claim. This probe would strengthen
it from "positions are public" to "a specific landmark trade is legible," which is
better but not different in kind.

---

## The closing paragraph the block asks for

Old section 1 asked what Hedera's institutional record demands that the hackathon
brief did not ask for. The record now answers it, and the answer is not the one
the plan anticipated.

> Hedera's institutional deployments are not withholding information. They are
> publishing an investor register, per-account compliance status, exact positions,
> and onboarding cohorts, on mainnet, under FCA-regulated issuance, for funds
> carrying the names of three of the largest asset managers in the world. Not one
> of the announcements mentions this, because it was never a decision. It is the
> token layer's default, and it is the default every future issuance inherits.
>
> So the thing the brief did not ask for is not a feature. It is a correction. A
> secondary market for ATS-issued assets, built the obvious way, publishes every
> participant's position to anyone who can run one HTTP request. The privacy layer
> is not an enhancement to that market. It is the precondition for the market
> being usable by the institutions Hedera has already onboarded.

That paragraph is a candidate for the README's second line, and unlike most pitch
copy it is checkable in about fifteen seconds by a judge with a terminal.
