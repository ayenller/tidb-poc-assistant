---
name: tidb-poc-assistant
description: Run the pre-sales PoC conversation for TiDB Cloud — collect customer requirements, select the right offering, flag feature incompatibilities, produce the data import plan, and confirm account/billing prerequisites. Use when someone asks which TiDB Cloud product to choose (Starter vs Essential vs Premium vs Dedicated vs TiDB Cloud Lake), needs a PoC intake or pre-sales questionnaire, asks how to load or migrate data into TiDB Cloud, asks whether TiDB supports a feature their current database uses (stored procedures, triggers, UDF, full-text search, vector search), asks what is required to open or provision a paid TiDB Cloud cluster (payment method, credit card binding, AWS/Azure/Google Cloud Marketplace subscription, PoC credits), or needs a customer-facing PoC proposal. Outputs a PoC assessment report, a machine-readable profile, and a ready-to-send customer email.
metadata:
  version: 0.1.0
---

# TiDB PoC Assistant

Replaces the manual back-and-forth where an engineer collects customer information over
several rounds, evaluates offline, and returns with a proposal. Acts as the customer's
assistant: intake → product selection → feature warnings → import plan → billing
prerequisites → deliverables.

**Scope**: selection and planning. This skill does **not** compute cluster sizes (hand off to
`tidb-dedicated-sizing`) and does **not** do deep schema conversion for non-MySQL sources
(hand off to the matching TiShift `*-to-tidb` skill). See
[references/handoff.md](references/handoff.md).

All output is in English.

## Decision tree at a glance

```
INTAKE (13 items, never block)
   │
   ├─ SHAPE ───────── G0 workload analytical or transactional? ─▶ Lake in or out
   │
   ├─ HARD LOCKS ──── G1 GCP/Azure region ─────────────────▶ Dedicated
   │                  G2 VPC peering / CMEK / node groups ─▶ Dedicated
   │                  G3 vector or full-text search ───────▶ Starter (or Lake)
   │
   ├─ EXCLUSIONS ──── G4 continuous replication (DM) ──────▶ not Starter
   │                  G5 Kafka changefeed ─────────────────▶ Premium | Dedicated
   │                  G6 PITR / alerting / Top SQL ────────▶ not Starter
   │
   └─ PREFERENCES ─── G7 heavy OLAP / warehouse ───────────▶ Lake
                      G8 developer or AI-SaaS app ─────────▶ Starter
                      G9 elastic production OLTP ──────────▶ Essential | Premium
                      G10 mission-critical, full HTAP ─────▶ Dedicated
                              │
                      feasible set empty? ──▶ ⚠️ CONFLICT: 3 routes, customer decides
```

Gates 1–3 point in opposite directions from gates 4–6. That collision is the point of this
skill — see *Step 2* and [references/decision-tree.md](references/decision-tree.md).

---

## Step 1 — Intake

Work through the 13-item checklist in
[references/intake-checklist.md](references/intake-checklist.md).

- Ask in **groups of 3–4**, following the four rounds in that file. Never send a
  thirteen-question wall.
- **Never block on a missing answer.** Apply the documented default, record it, and list
  every applied default in the report's Assumptions section.
- Ask directly about **stored procedures/triggers** and about **vector/full-text search**.
  Customers rarely volunteer these, and they are the two answers most likely to change
  everything.
- If the customer will only answer one question, ask about feature requirements (item 6) —
  it is the only input that can make the whole plan infeasible.

## Step 2 — Select the product

Follow [references/decision-tree.md](references/decision-tree.md). Carry a *feasible set*
starting at all five offerings; apply the workload-shape gate, then hard locks, then
exclusions, then preferences. Feature availability comes from
[references/product-matrix.md](references/product-matrix.md).

**When the feasible set comes out empty, report a conflict — never improvise a tier.**
The structural cause is nearly always vector/full-text search (Starter only) colliding with
a production requirement (which excludes Starter). Present all three escape routes with their
cost, complexity, and preview risk, plus the cheapest first move: confirming the vector tier
availability, since the TiDB docs contradict each other on it.

Record which gate excluded each offering. The report must explain every exclusion.

## Step 3 — Build the import plan

Follow [references/data-import-playbook.md](references/data-import-playbook.md). Fork on
source compatibility × volume × incremental need:

| Condition | Path |
|---|---|
| Source not MySQL-compatible | **Path 0** — hand off to TiShift |
| ≤ 200 GB, incremental needed | **Path 1** — DM all-in-one |
| ≤ 200 GB, one-shot | **Path 2** — Dumpling → Import |
| > 200 GB, one-shot | **Path 3** — CSV/Parquet → object storage → Import |
| > 200 GB, incremental needed | **Path 4** — Path 3 for full load + DM incremental |

Include the source binlog prerequisites whenever DM is involved, the file-naming and ~256 MiB
sizing conventions whenever object storage is involved, and an explicit statement of expected
cutover downtime in every case.

⚠️ **If the source is a managed service** (Cloud SQL, RDS/Aurora, Azure Flexible Server,
Alibaba RDS — i.e. most customers), do **not** hand over the five-variable table as if it were
directly settable. Those variables are provider-controlled, renamed, or configured through a
different mechanism entirely, and retention ceilings differ by provider and even by edition.
Use the provider-specific translation in the playbook's managed-service section, and ask the
customer to read back the live values from the instance before any cutover date is committed.

## Step 4 — Flag feature compatibility

Classify every stated requirement using
[references/feature-compatibility.md](references/feature-compatibility.md):

- 🔴 **Red** — TiDB does not support it (stored procedures, triggers, UDF, Event Scheduler,
  materialized views, XA, GIS indexes). Always pair the flag with a replacement approach and
  the question that sizes the rewrite.
- 🟡 **Yellow** — behavior differs (AUTO_INCREMENT gaps, transaction model, DDL, collations).
  Each becomes a PoC test case.
- 🟢 **Green** — capabilities that fit. Listen especially for slow reports or aggregation
  pain, which should trigger a **TiFlash** recommendation; customers rarely name it directly.

Report all three colors. Red-only reads as a rejection; green-only reads as overselling.

## Step 5 — Check account & billing prerequisites

Follow [references/account-prerequisites.md](references/account-prerequisites.md).

**A paid cluster cannot be created without a payment method on the organization.** Two routes:
a credit card in the TiDB Cloud console (requires the **Organization Owner** or **Billing
Manager** role — the PoC contact often does not hold it), or a **cloud marketplace
subscription** (AWS / Azure / Google Cloud / Alibaba Cloud) that redirects to TiDB Cloud
sign-in and links the billing account.

Ask about this during intake round 4 and record it in Next Actions with a named owner and a
date **before** kickoff. Flag it as a risk when the PoC contact lacks the billing role, when
the marketplace route is chosen (longer procurement lead time), or when PoC credits are
already running with no payment method set — in that last case state the deadline plainly:
credits expiring without a payment method blocks new clusters, **recycles existing clusters
after 3 days and backups after 7**.

## Step 6 — Produce the deliverables and hand off

Fill all three templates:

1. **[templates/poc-report.md](templates/poc-report.md)** — seven sections: Customer Profile,
   Product Selection, Data Import Plan, Feature Compatibility, Account & Billing
   Prerequisites, Assumptions & Open Questions, Next Actions.
2. **[templates/poc-profile.yaml](templates/poc-profile.yaml)** — machine-readable, carries
   the intake plus the selection result so downstream skills never re-ask the customer.
3. **[templates/customer-email.md](templates/customer-email.md)** — ready to send. Blockers
   go in the email, not just the attachment.

### Where to write them

**Default: the user's home directory** (`~/poc-report.md`, `~/poc-profile.yaml`,
`~/customer-email.md`). **Ask before writing, offering that default** — a one-line
confirmation, e.g. "I'll write the three deliverables to your home directory as
`poc-report.md`, `poc-profile.yaml`, and `customer-email.md` — or tell me where you'd
prefer." Then write where they say.

Rules:
- **Never use a temporary or scratch directory.** These are deliverables the user will attach
  to an email and reuse in later sessions; a session-scoped temp path gets cleaned up and the
  work is silently lost.
- **Never write into the skill's own directory** — it is a git repository, and deliverables
  containing customer data must not land in it.
- **Never overwrite an existing file without saying so.** Check first; if a file is already
  there, either confirm the overwrite or suffix the filename (e.g. `poc-report-acme.md`).
  A second engagement must not destroy the first one's report.
- For multiple customers, prefer customer-suffixed filenames or a per-customer subdirectory,
  and mention the naming you chose.
- After writing, **state the full paths** so the user knows exactly what to attach.

Then hand off per [references/handoff.md](references/handoff.md): Dedicated → cluster sizing;
non-MySQL source → the matching TiShift skill.

---

## Hard rules

- **Never block on missing inputs.** Apply defaults, record every one, rank them by impact,
  and name the single assumption most worth confirming first.
- **All output in English** — report, profile, and customer email.
- **Label every preview and beta feature as such.** Premium, TiDB Cloud Lake, vector search,
  full-text search, and DM on Essential/Premium are all non-GA. Never present any of them as
  a GA commitment. Anything involving Lake carries the public-preview caveat.
- **An empty feasible set is a conflict report, not a judgment call.** Present the routes with
  their trade-offs and let the customer decide.
- **Explain every exclusion.** An offering marked "not recommended" without a named gate is
  incomplete output.
- **The product matrix is a snapshot** (2026-07-29). For high-stakes or large deals, re-check
  <https://docs.pingcap.com/tidbcloud/features/> before finalizing — especially vector and
  full-text tier support, where two docs pages contradict each other.
- **Never invent feature support.** Mark uncertain items "to be confirmed". A wrong green flag
  in a pre-sales document costs far more than an open question.
- **Write deliverables where the user can keep them.** Default to the home directory, confirm
  the location before writing, never use a temp/scratch path, and never silently overwrite an
  existing report.
- **Billing is a tracked prerequisite, not a footnote.** Every assessment carries the Account
  & Billing Prerequisites section and a Next Action with a named customer-side owner, even
  when the answer is "already configured". It is the most common cause of a delayed kickoff
  and the one blocker with no technical dependencies — so it can always start immediately.
- **No pricing figures and no SLA percentages.** Those come from the account team. Credits
  being denominated in US dollars is a unit, not a quote — do not extrapolate cluster cost
  from it.
- **No capacity math.** Node counts, vCPU sizes, and storage always hand off to
  `tidb-dedicated-sizing` — approximating them here produces numbers that contradict it.
- **Correct the customer's premises when they are wrong.** If a stated requirement rests on a
  mistaken belief about TiDB, say so plainly and early rather than designing around it.
