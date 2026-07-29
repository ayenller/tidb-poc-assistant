# TiDB PoC Assessment — {{Customer Name}}

> Template. Seven sections, in this order, every time. Delete the guidance in `<!-- -->`
> comments when filling it in. Keep a section even when it is empty — write "None
> identified" rather than dropping it, so the reader knows it was considered.

**Date**: {{YYYY-MM-DD}} · **Prepared by**: {{name}} · **Source database**: {{type + version}}

---

## 1. Customer Profile

<!-- The intake, echoed back. This section exists so the customer can correct a wrong input
     before it propagates into a wrong recommendation. Mark every assumed value. -->

| # | Item | Value |
|---|---|---|
| 1 | Source database & version | |
| 2 | Data volume (raw) / largest table | |
| 3 | Peak QPS / TPS, read:write | |
| 4 | Latency SLA | |
| 5 | Continuous replication required | |
| 6 | Feature requirements | |
| 7 | Workload profile | |
| 8 | Cloud provider & region | |
| 9 | Network & compliance | |
| 10 | Tolerable cutover window | |
| 11 | PoC goals & timeline | |
| 12 | Team profile | |
| 13 | Account & billing readiness | |

Values marked *(assumed)* were not provided — see section 6.

---

## 2. Product Selection

### Recommendation: **{{Starter / Essential / Premium / Dedicated / TiDB Cloud Lake}}**

<!-- One offering, named. Then the reasoning, quoting the customer's own words. -->

**Why**

| Gate | Your requirement | Effect |
|---|---|---|
| {{G#}} | {{quote the stated requirement}} | {{what it locked or excluded}} |

**Why not the others** <!-- One line each. Never leave an offering unexplained. -->

| Offering | Not recommended because |
|---|---|
| Starter | |
| Essential | |
| Premium | |
| Dedicated | |
| TiDB Cloud Lake | |

**Preview / beta dependencies** <!-- Every non-GA feature the recommendation relies on.
Omit only if there are genuinely none. -->

| Feature | Status | What that means for the PoC |
|---|---|---|

### ⚠️ Requirement Conflict

<!-- Include ONLY when the feasible set came out empty. Delete the whole subsection
     otherwise. Never resolve the conflict unilaterally — present the routes and let the
     customer choose. -->

**The conflict**: {{requirement A}} requires {{offering X}}, while {{requirement B}} rules
{{offering X}} out. No single offering satisfies both.

**First, the cheapest check**: {{when the conflict involves vector search — the TiDB docs
contradict each other on tier availability. Confirm with the account team before redesigning;
the conflict may not be real.}}

| Route | Approach | Cost | Complexity | Risk |
|---|---|---|---|---|
| A | Split clusters | | | |
| B | TiDB Cloud Lake | | | |
| C | Externalize search | | | |

**Recommendation for choosing**: {{which route fits this customer's stated priorities, and
what fact would change the answer}}

---

## 3. Data Import Plan

**Path**: {{Path 0–4}} — chosen because {{volume}} and {{replication requirement}}.

**Expected downtime at cutover**: {{be explicit — this is the number the customer's business
side cares about}}

### Prerequisites

<!-- For DM paths, include the binlog table verbatim. -->

| Setting | Required value | Current value | Action |
|---|---|---|---|

### Steps

1.
2.

### Network

{{Public IP / Private Link / VPC Peering — and who needs to action it}}

### Pitfalls to avoid

-

---

## 4. Feature Compatibility

### 🔴 Not supported — replacement required

| Feature | Used for | Replacement approach | Effort to estimate |
|---|---|---|---|

### 🟡 Behavior differences — verify during the PoC

| Area | What differs | PoC test case |
|---|---|---|

### 🟢 Capabilities that fit

| Your requirement | TiDB capability |
|---|---|

<!-- If the source is not MySQL-compatible, state here that this is a first pass and the
     authoritative assessment comes from the matching TiShift skill. -->

---

## 5. Account & Billing Prerequisites

<!-- A paid cluster cannot be provisioned without a payment method on the organization.
     This section exists so provisioning never becomes the thing that delays kickoff.
     Keep it even when everything is already in place — write "already configured". -->

**Status**: {{Already configured / Card to be added / Marketplace subscription in progress /
Not started}}

**Route**: {{Credit card in the TiDB Cloud console — or — <Provider> Marketplace subscription}}

| Item | Status | Owner | Needed by |
|---|---|---|---|
| TiDB Cloud organization exists | | | |
| PoC contact holds Organization Owner or Billing Manager | | | |
| Payment method configured | | | |

<!-- For the marketplace route, state both consequences explicitly:
     1. Linking replaces any existing payment method on the organization.
     2. Marketplace-billed orgs cannot add payment methods or download invoices in the
        TiDB Cloud console — invoicing runs through the cloud provider. -->

**PoC credits**: {{granted? amount? expiry date?}}

> ⚠️ **Include this warning whenever credits are running with no payment method set.**
> If credits expire before a payment method is added, new Dedicated clusters cannot be
> created, **existing clusters are recycled after 3 days, and backups after 7 days.**

<!-- If the recommendation is Starter, note that its free allowance may cover the PoC without
     billing setup — but say so explicitly, and note that moving off Starter later requires
     it. Do not let the customer infer it. -->

---

## 6. Assumptions & Open Questions

### Assumptions applied for missing inputs

| Input | Assumed value | Impact if wrong |
|---|---|---|

<!-- Rank by impact. Name the single assumption most worth confirming first. -->

**Confirm this one first**: {{the assumption whose correction would most change the plan}}

### Open questions for the customer

1.

---

## 7. Next Actions

| # | Action | Owner | Depends on |
|---|---|---|---|
| 1 | Configure payment method (see section 5) | *customer* | — |

<!-- Keep the billing action in this table with a real owner and date. It has no technical
     dependencies, so it can start immediately and should never be what blocks kickoff. -->


**Handoffs**

- [ ] Cluster sizing → `tidb-dedicated-sizing` <!-- when the recommendation is Dedicated;
      list the input block -->
- [ ] Migration assessment → `TiShift/{{source}}-to-tidb` <!-- when the source is not
      MySQL-compatible -->

---

*This assessment is a PoC starting point based on the information provided. Node counts,
specifications, pricing, and SLAs are out of scope here. Feature availability marked preview
or beta may change — confirm current status before finalizing the architecture.*
