# Account & Billing Prerequisites

## Applicability

This reference covers database-cluster prerequisites. Lake and Filesystem require their
own billing/access verification; database payment, credit expiry, and cluster/backup
recycling rules must not be copied to them. Filesystem does not require creating a database.
BYOC additionally requires coordinated onboarding, cloud preparation, and permissions;
confirm commercial/account prerequisites with the TiDB team rather than assuming ordinary
self-service provisioning. See `product-matrix.md` for the dated sources.


Source: <https://docs.pingcap.com/tidbcloud/tidb-cloud-billing/>

A paid TiDB Cloud cluster **cannot be created without a payment method on the organization**.
This is the most commonly overlooked PoC blocker: the technical plan is agreed, the kickoff is
scheduled, and then provisioning stalls for days waiting on their procurement or
cloud-account team. Raise it during intake, not at kickoff.

There are two routes. The customer picks **one**.

---

## Route 1 — Credit card in the TiDB Cloud console

Direct card binding. Fastest when they can add a card themselves.

**Who can do it**: only the **Organization Owner** or **Organization Billing Manager** role.
Confirm they actually hold one of these — a developer running the PoC
usually does not, and this is a frequent source of delay.

**Two entry points**
1. **During cluster creation** — click **Add Credit Card** on the Create Resource page and
   fill in the card details.
2. **Ahead of time (recommended)** — TiDB Cloud console → **Billing** → **Payment Method**
   tab → **Add a New Card**.

Multiple cards can be stored, with one designated as the default for automatic billing.

**Recommend doing this before the PoC kickoff**, not during. It removes provisioning from the
critical path.

---

## Route 2 — Cloud marketplace subscription

The customer subscribes to TiDB Cloud through their cloud provider's marketplace and pays via
their existing cloud account. Charges appear on their cloud bill.

**Available on**: AWS Marketplace, Azure Marketplace, Google Cloud Marketplace, Alibaba Cloud
Marketplace.

**Flow**: search for *TiDB Cloud* in the marketplace → subscribe → follow the on-screen
instructions, which redirect to TiDB Cloud sign-in → complete account setup / linking.

**Linking an existing TiDB Cloud account**: if they already have a TiDB Cloud account
and wants to pay through their cloud billing account, they subscribe in the marketplace and
then sign in with the existing TiDB Cloud account. A **Link to Your \<Provider\> Billing
Account** page appears — select the target organization and click **Link**.

⚠️ **Two consequences to state explicitly when recommending this route:**

1. **Linking replaces the existing payment method.** Per the docs: "If your organization
   already has a payment method in TiDB Cloud, the existing payment method for this
   organization will be replaced by the newly added billing account." If they already
   have a card on file, tell them it will be superseded.
2. **Marketplace-billed organizations lose console billing functions.** Per the docs, you
   "cannot add payment methods or download invoices in the TiDB Cloud console." Invoicing runs
   through the cloud provider. Customers whose finance team expects TiDB Cloud invoices need
   to know this before they subscribe, not after.

**When to prefer this route**: they have committed cloud spend (EDP / MACC / committed
use discounts) they want to draw down against, procurement is easier through an existing cloud
vendor relationship, or adding a card requires an approval cycle they'd rather avoid.

---

## PoC credits — and the deadline they create

TiDB Cloud grants credits to PoC users; **one credit equals one US dollar**. Credits are
consumed **before** any payment method is charged, so a PoC can often run entirely on credits.

The [billing page](https://docs.pingcap.com/tidbcloud/tidb-cloud-billing/) was rechecked
2026-10-30. Its warning is specifically about Dedicated: expired credits without a payment
method prevent new Dedicated clusters; existing Dedicated clusters are recycled after
3 days and backups after 7. Do not extend that timeline to other products. Verify the
applicable policy again before quoting a future deadline.

Credits cannot pay support-plan fees. BYOC requires Enterprise support or higher, with
cloud-provider infrastructure charges and PingCAP service/support charges to confirm;
these prerequisites differ from a normal self-service plan. No price estimate is made here.

---

## What to do in the assessment

Treat this as a **next action with an owner and a deadline**, never as a footnote.

**Ask during intake** (fold into round 4, the project round):
- Do they already have a TiDB Cloud account and organization?
- Does the PoC contact hold Organization Owner or Billing Manager, or does someone else?
- Card or marketplace? If marketplace — which provider, and does committed cloud spend exist?
- Is there a PoC credit grant, and when does it expire?

**Record in the report's Next Actions** with a named owner on their side and a date
ahead of the PoC kickoff.

**Flag as a risk** whenever any of these hold:
- The PoC contact lacks the billing role → an approval chain exists that nobody has started.
- Marketplace subscription chosen → procurement lead time is typically longer than card entry.
- Credits are already running with no payment method set → verify the product-specific
  expiry and retention policy, then record the actual deadline.

**Note the interaction with plan selection**: billing readiness is broadly relevant,
but the quoted recycling timeline is Dedicated-specific. MySQL-compatible Starter has a
free allowance, so a Starter-only PoC may not need billing set up on day one — but say so
explicitly rather than leaving them to assume it, and point out that moving off
Starter later will require it.

---

## What not to state

- **No pricing figures.** Credits are denominated in dollars, which is a unit, not a quote.
  Cluster cost estimates come from their TiDB contact.
- Do not advise on which route is cheaper — that depends on their cloud contracts.
  Present the trade-offs and let them choose.
- Do not walk a customer through entering card details in a live conversation.
