# V2. Kinexys (J.P. Morgan)

**State:** filled, and most cells are `unknown by construction`. That is the
finding, not a gap in the reading.
**Closed by:** J.P. Morgan's own Kinexys Digital Financing and asset tokenization
pages, plus reporting on the intraday repo launch.

The honest rival design, and the one a judge raises. Intraday repo against
tokenised collateral is our use case, already in production, with privacy solved
by permissioning instead of cryptography.

---

## What it is, stated so the comparison is fair

Digital Financing on Kinexys provides secured financing through the exchange of
cash for tokenised collateral, with delivery versus payment enabling
"near-simultaneous transfer of cash and collateral ownership." Cash and collateral
are tokenised as smart contracts before execution. Repo traders exchange cash held
at J.P. Morgan against collateral at HQLAx intraday, with settlement and maturity
specified to the minute, unwinding within hours rather than a one to two day
cycle.

Access requires a J.P. Morgan banking relationship and approval as an eligible
institutional participant. The platform does not support retail clients,
permissionless token trading, or unregulated digital assets. The network is
private and permissioned.

**Do not understate it.** This is our use case, working, at scale, with real
counterparties and real intraday settlement. Our advantage is not that we do repo
better. It is narrower than that, and the row below is where it lives.

---

## The nine rows

| F4 row | G | O | T | Mechanism |
|---|---|---|---|---|
| Trader identity | full | J.P. Morgan, plus the counterparty | at trade | permissioning |
| KYC credential | full | J.P. Morgan | at onboarding | permissioning |
| Order size | unknown | admitted participants at most | unknown | permissioning |
| Order price | unknown | admitted participants at most | unknown | permissioning |
| Execution price | unknown | admitted participants at most | unknown | permissioning |
| Quantity | unknown | admitted participants at most | unknown | permissioning |
| Asset | full | participants | at trade | permissioning |
| Compliance validity | asserted, not published | participants, by inference | at trade | permissioning |
| Settlement validity | asserted, not published | participants | at settlement | permissioning |

**Every `unknown` here has the same cause and it is a single fact:** the observer
set is bounded by the guest list, so no cell needs an individual answer. Once you
are outside the network you learn nothing, and once you are inside, what you learn
is whatever J.P. Morgan's implementation shows you, which is not published.

F3b ruled in advance that `unknown` on a permissioned venue is itself the finding.
It is worth stating why, precisely: **a permissioned venue does not have a
disclosure policy. It has an admission policy, and a disclosure policy is what you
need when admission is not enough.** That is the substantive difference between
permissioning and the other three mechanisms, and it is why permissioning is
listed in F3b's taxonomy as the mechanism whose cost is that "verification
collapses into trusting the operator."

---

## The two questions

**What is hidden, from whom, until when, by which mechanism?**

Everything, from everyone outside the network, indefinitely, by permissioning.
Inside the network, unspecified in public material.

**What does the venue still prove while that information is hidden, and who has
to be trusted?**

Nothing, to anyone, without trusting J.P. Morgan. That is not a criticism of the
engineering. It is the structural consequence of the mechanism: the ledger is
operated by a party that is also a counterparty, a custodian, and the cash issuer.
A participant's assurance that a counterparty is eligible, that collateral exists,
and that settlement was atomic is J.P. Morgan's assurance. There is no artefact a
participant can check independently, and there is nothing at all for a
non-participant, a supervisor with a different remit, or a future auditor.

---

## The sentence this row exists to produce

DISCLOSURE-LATTICE V flags the line distinguishing our matrix from a permissioned
venue as "the single most contested sentence in the pitch." Here it is, in the
form this row supports:

> A Kinexys participant cannot verify that their counterparty is eligible. They
> can only rely on J.P. Morgan having checked. We make eligibility publicly
> verifiable **without disclosing who the counterparty is**, which is the one
> combination permissioning cannot produce, because permissioning hides by
> restricting the audience and a restricted audience cannot verify anything for
> anyone outside it.

Note what the sentence does not claim. It does not claim we are more private.
Kinexys is more private against an outside observer, trivially, because there is
no outside observer. It claims something orthogonal and defensible: privacy from
permissioning and public verifiability are mutually exclusive, and we are on the
other side of that trade.

---

## Honest limits of this row

- The public material describes a **product**, not a protocol. No message spec, no
  contract, no ledger visibility statement. The cells above are inferred from the
  admission model, which is the only mechanism the sources actually state.
- Whether an internal participant sees another's book is genuinely unknown and is
  recorded as unknown. It is not knowable from published sources, and guessing it
  would be exactly the "plausible cell" F3b forbids.
- If Kinexys has published a technical specification we have not found, this row
  changes. It is the row most likely to be wrong, and it is the row a judge is
  most likely to know about, so it should be re-checked before the pitch.
