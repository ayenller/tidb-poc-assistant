# Intake Checklist and Defaults

Thirteen items. Items 1–9 are **hard inputs** — they drive gates in `decision-tree.md` or the
fork in `data-import-playbook.md`. Items 10–13 shape the plan and the report.

**Never block on a missing input.** Apply the default, record it, and list every applied
default in the report's "Assumptions & Open Questions" section.

## How to ask

Ask in **groups of 3–4**, not as a thirteen-question wall. Customers abandon questionnaires.
Suggested grouping:

1. **Round 1 — the shape of the thing**: source DB type & version, data volume, cloud +
   region. (Items 1, 2, 8 — this round alone often fires G1 and decides the import fork.)
2. **Round 2 — the workload**: peak QPS/TPS, read/write ratio, latency SLA, workload profile.
   (Items 3, 4, 7)
3. **Round 3 — the constraints**: continuous replication, feature requirements, network &
   compliance. (Items 5, 6, 9 — this round decides most conflicts.) If the customer is
   multi-tenant or SaaS, ask directly whether one tenant's workload can be allowed to affect
   the others — they will not volunteer it as a requirement.
4. **Round 4 — the project**: cutover window, PoC goals and timeline, team profile, and
   **account & billing readiness**. (Items 10, 11, 12, 13 — item 13 is the one that most
   often delays a kickoff, so do not skip it because it feels commercial rather than
   technical.)

If the customer volunteers information out of order, take it and skip ahead. If they answer
everything in one paragraph, confirm the gaps and move on.

## Checklist

| # | Input | Drives | Default when missing |
|---|---|---|---|
| 1 | **Source database type & version** (MySQL, Aurora, PostgreSQL, Oracle, SQL Server, MongoDB, Spanner, CockroachDB, …) | Import path; TiShift handoff; compatibility red flags | **MySQL-compatible** — state the assumption prominently, since a non-MySQL source changes the entire import plan |
| 2 | **Total raw data volume** (uncompressed, data + indexes), plus largest single table | **The 200 GB import fork**; sizing handoff | **Unknown → assume > 200 GB** and present the object-storage import path; note that under 200 GB a simpler path exists |
| 3 | **Peak QPS / TPS**, read/write ratio | Sizing handoff only | **None assumed** — pass through as unknown; `tidb-dedicated-sizing` has its own defaults |
| 4 | **Latency SLA** (P95/P99 target) | Sizing handoff only | **None assumed** — pass through as unknown |
| 5 | **Continuous replication from production?** (ongoing sync vs one-shot cutover) | **G4 — excludes Starter**; DM section of the import plan | **Assume yes** for any production migration — it is the safer assumption, and getting it wrong the other way produces an unusable plan. Confirm explicitly. |
| 6 | **Feature requirements**: UDF, Trigger, Stored Procedure, Event Scheduler, materialized views, XA, foreign keys, GIS/spatial, **full-text search**, **vector search** | **G3 hard lock**; red/yellow flags in `feature-compatibility.md` | **None assumed** — but always ask directly about stored procedures/triggers and about vector/full-text; customers rarely volunteer these and they are the highest-impact answers |
| 7 | **Workload profile**: pure OLTP / OLTP + reporting (HTAP) / heavy OLAP or warehouse | G7 (Lake); TiFlash recommendation | **Pure OLTP** — no TiFlash; note it can be added online later |
| 8 | **Cloud provider + region** | **G1 hard lock** | **Unspecified** — state that GCP or Azure would lock Dedicated, and ask before finalizing |
| 9 | **Network & compliance**: VPC peering, private endpoint, CMEK, database audit logging, data residency, **tenant/workload isolation** | **G2 hard lock**; DM network options | **None assumed** — but ask about VPC peering specifically for any enterprise customer, and about tenant isolation for any multi-tenant or SaaS customer |
| 10 | **Tolerable cutover window** (acceptable downtime) | One-shot import vs DM-based near-zero-downtime cutover | **Assume a short window is required** → plan for DM-based cutover |
| 11 | **PoC goals, success criteria, timeline** | Report action items; scope of the PoC | **Unstated** — list "define PoC success criteria" as the first action item |
| 12 | **Team profile**: developers / AI-SaaS / traditional enterprise IT | G8 weighting toward Starter | **Unspecified** — no weighting applied |
| 13 | **Account & billing readiness**: existing TiDB Cloud org? does the PoC contact hold Organization Owner / Billing Manager? card or marketplace (which provider)? PoC credits and their expiry? | Provisioning prerequisite — see `account-prerequisites.md` | **Assume nothing is set up** and list "confirm payment method before kickoff" as a next action with a customer-side owner |

## Normalization

- Treat customer-quoted "TB" as TiB (1 TB = 1,024 GiB) unless they clearly mean decimal;
  record the interpretation.
- TPS → QPS: multiply by statements per transaction; if unknown assume 5 and record it.
- If the volume quoted includes replication (Aurora cluster storage, MongoDB replica-set
  total), divide back to a single logical copy before applying the 200 GB fork.
- "Compressed size" from the source is not the right input — the fork and sizing both use
  **raw uncompressed** volume. If only a compressed figure is available, say so in the
  assumptions.

## Inputs outside the checklist

Anything else the customer offers — source-side CPU/RAM utilization, instance topology,
connection-pool settings, current pain points — is **context**. Use it to sharpen risk notes,
PoC validation items, and the customer email. It must not silently change the tier
recommendation or the import path; if it seems like it should, say so explicitly and explain
why rather than adjusting quietly.

## Minimum viable intake

If the customer will only answer one question, ask **item 6** (feature requirements) — it is
the only input that can make the entire plan infeasible. Second priority: **item 8**
(cloud/region), then **item 2** (volume).

Item 13 is different in kind: it cannot make the plan infeasible, but it is the most common
cause of a *delayed* PoC. Ask it even in a short conversation — the answer takes one sentence
and it is the difference between provisioning on day one and waiting on someone's procurement
team.
