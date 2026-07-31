# TiDB Cloud PoC Assessment — {{your company / system}}

> Template. Seven sections, in this order, every time. Delete the guidance in `<!-- -->`
> comments when filling it in. Keep a section even when it is empty — write "None
> identified" rather than dropping it, so the reader knows it was considered.

**Date**: {{YYYY-MM-DD}} · **Source database**: {{type + version}}

> Your own working document. Share it with your architects, your management, or your TiDB
> contact — it is written to survive being quoted back at you.

---

## 1. Your Environment

<!-- The intake, echoed back, so a wrong input can be corrected before it propagates into a
     wrong recommendation. Mark every assumed value. -->

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
| 14 | Data masking / PII required | |

Values marked *(assumed)* were not provided — see section 6.

---

## 2. Product Selection

### Recommendation: **{{Starter / Essential / Premium / Dedicated / TiDB Cloud Lake}}**

<!-- One offering, named. Then the reasoning, quoting their own words back. -->

**Why**

| Gate | Your requirement | Effect |
|---|---|---|
| {{G#}} | {{quote their stated requirement}} | {{what it locked or excluded}} |

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
     otherwise. Never resolve the conflict unilaterally — present the routes and let them
     choose; it is their architecture. -->

**The conflict**: {{requirement A}} requires {{offering X}}, while {{requirement B}} rules
{{offering X}} out. No single offering satisfies both.

**First, the cheapest check**: {{when the conflict involves vector search — the TiDB docs
contradict each other on tier availability. Ask your TiDB contact to confirm before
redesigning anything; the conflict may not be real.}}

| Route | Approach | Cost | Complexity | Risk |
|---|---|---|---|---|
| A | Split clusters | | | |
| B | TiDB Cloud Lake | | | |
| C | Externalize search | | | |

**Which to choose**: {{which route fits their stated priorities, and what single fact would
change the answer}}

---

## 3. Data Import Plan

**Path**: {{Path 0–4}} — chosen because {{volume}} and {{replication requirement}}.

**Expected downtime at cutover**: {{be explicit — this is the number the business side will
ask about}}

### Prerequisites

<!-- For DM paths, include the binlog requirements table. If the source is a managed service
     (Cloud SQL / RDS / Azure / Alibaba), give the provider-specific translation too — those
     variables are not directly settable, and retention ceilings differ by provider and
     edition. Fill in "Current value" from the live instance, not from defaults. -->

| Setting | Required value | Current value | Action |
|---|---|---|---|

### Steps

1.
2.

### Network

{{Public IP / Private Link / VPC Peering — and who needs to action it}}

### Pitfalls to avoid

-

### Data Masking

<!-- Include when masking is required (item 14); delete the subsection otherwise.
     Full guidance in references/data-masking.md. -->

**Approach**: AWS DMS native data masking, applied in flight. The source database is not
modified and plaintext never lands on disk or in a log.
**Route**: {{s3 (recommended — masked values can be verified before they reach the cluster) /
direct}}

> ⚠️ **Masking is full-load only — there is no CDC.** {{State what this means for the cutover
> plan. If a near-zero-downtime cutover was also required, this is a conflict: see below.}}

**Per-column feasibility** <!-- Whether a column can be masked depends on the type DMS maps it
to, not on the MySQL type name. Ask for exact types. -->

| Column | MySQL type | Class | Action | Note |
|---|---|---|---|---|
| | | A / B / C | hash / digits-mask / digits-randomize / drop / keep | |

<!-- Class A = fully maskable (VARCHAR, CHAR, TEXT, ENUM…).
     Class B = digits-randomize only, which preserves length and format — weak.
     Class C = cannot be masked: JSON, MEDIUMTEXT, LONGTEXT, BLOB, DATE, DATETIME, FLOAT,
               DOUBLE. Drop the column or change the source type.
     Traps worth naming: TEXT is maskable but MEDIUMTEXT/LONGTEXT are not; dates cannot be
     masked, so date of birth needs a decision; FLOAT/DOUBLE cannot be masked. -->

**Sensitive columns that cannot be fully masked**: {{list each with its three options — drop,
change source type, or accept weak/no masking. If empty, write "None".}}

**Prerequisite**: DMS engine version **≥ 3.5.4**. Below this, masking rules are accepted and
then silently ignored — plaintext reaches the target with no error anywhere.

**How masking will be proven**: {{row counts, plaintext residue scan, and exact-value
comparison — DMS hash-mask equals UPPER(SHA2(value,256)), so every masked cell is checkable
against the source. On the S3 route this happens before anything is loaded.}}

**Target schema consequences**: {{hashed columns widen to CHAR(64); collation remapping and
its effect on sort/comparison semantics; foreign keys and FULLTEXT/SPATIAL indexes omitted.}}

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
| Someone on your side holds Organization Owner or Billing Manager | | | |
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
     it. Do not leave it to be inferred. -->

---

## 6. Assumptions & Open Questions

### Assumptions applied for missing inputs

| Input | Assumed value | Impact if wrong |
|---|---|---|

<!-- Rank by impact. Name the single assumption most worth confirming first. -->

**Confirm this one first**: {{the assumption whose correction would most change the plan}}

### Open questions to take to your TiDB contact

1.

---

## 7. Next Actions

| # | Action | Owner | Depends on |
|---|---|---|---|
| 1 | Configure payment method (see section 5) | *you / your billing owner* | — |

<!-- Keep the billing action in this table with a real owner and date. It has no technical
     dependencies, so it can start immediately and should never be what blocks kickoff. -->


**Handoffs**

- [ ] Cluster sizing → `tidb-dedicated-sizing` <!-- when the recommendation is Dedicated;
      list the input block. Node counts and specs are out of scope here. -->
- [ ] Migration assessment → `TiShift/{{source}}-to-tidb` <!-- when the source is not
      MySQL-compatible -->

---

*This assessment is a starting point based on the information provided, not a vendor
commitment. Node counts, specifications, pricing, and SLAs are out of scope — get those from
your TiDB contact. Feature availability marked preview or beta may change; confirm current
status at <https://docs.pingcap.com/tidbcloud/features/> before committing to an
architecture.*
