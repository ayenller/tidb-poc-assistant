# tidb-poc-assistant

A portable, agent-agnostic skill that runs the **pre-sales PoC conversation for TiDB Cloud**.

It replaces the manual loop where an engineer collects customer information over several
rounds, evaluates offline, and comes back with a proposal. Instead, the agent acts as the
customer's assistant and drives the whole thing in one conversation: requirement intake →
product selection → feature-compatibility warnings → data import plan → account & billing
prerequisites → deliverables.

Pure Markdown — no scripts, no agent-specific frontmatter beyond `name` and `description` —
so it works with Claude Code and any agent supporting the open Agent Skills format.

## Layout

```
tidb-poc-assistant/
├── SKILL.md                          # Entry point: 6-step workflow, decision tree, hard rules
├── README.md
├── references/
│   ├── intake-checklist.md           # 13-item checklist, ask-in-rounds guidance, defaults
│   ├── decision-tree.md              # 10 ordered gates + conflict arbitration + mermaid map
│   ├── product-matrix.md             # features-page snapshot, Lake, the docs contradiction
│   ├── data-import-playbook.md       # 5 import paths, binlog prereqs, CSV conventions
│   ├── feature-compatibility.md      # red / yellow / green classification + workarounds
│   ├── account-prerequisites.md      # payment method, marketplace subscription, PoC credits
│   └── handoff.md                    # input contracts for downstream skills
└── templates/
    ├── poc-report.md                 # 7-section assessment report
    ├── poc-profile.yaml              # machine-readable profile (schema v1)
    └── customer-email.md             # ready-to-send email, with conflict variants
```

## Install

**Claude Code (personal):**

```bash
ln -s "$PWD/tidb-poc-assistant" ~/.claude/skills/tidb-poc-assistant
# or: cp -R tidb-poc-assistant ~/.claude/skills/
```

**Claude Code (project):** copy into `.claude/skills/` in the repo.
**Other agents:** point the agent at `SKILL.md`; `references/` and `templates/` must be
readable from the same directory.

## Usage

Describe the customer's situation in natural language:

> A customer runs MySQL 8.0, about 800 GB, peak 30k QPS, needs continuous sync from
> production during migration, on AWS Tokyo. Which TiDB Cloud product, and how do we load
> the data?

Or start from nothing and let the skill interview you — it asks in rounds of 3–4 questions
rather than presenting a thirteen-item form.

Output: a seven-section assessment report, a `poc-profile.yaml`, and a customer email.

## The decision tree

Eleven gates in four phases, carrying a feasible set that starts at all five offerings:

| Phase | Gates | Effect |
|---|---|---|
| **Shape** | G0 workload analytical vs transactional | Decides whether Lake is a candidate |
| **Hard locks** | G1 cloud region · G2 enterprise infra/compliance · G3 vector or full-text search | Reduce the set to one |
| **Exclusions** | G4 continuous replication (DM) · G5 Kafka changefeed · G6 production observability | Remove members |
| **Preferences** | G7 warehouse · G8 developer/AI-SaaS · G9 elastic OLTP · G10 mission-critical | Rank survivors |

**The conflict this exists to catch**: vector and full-text search are available on **Starter
only**, while DM (continuous replication from production) is available on **everything except
Starter**. A customer wanting both has no single offering that works. The skill detects the
empty feasible set, refuses to improvise a tier, and presents three escape routes — split
clusters, TiDB Cloud Lake, or externalized search — with cost, complexity, and preview risk
for each.

## Key rules encoded in the skill

- **Never block on missing inputs** — apply documented defaults, list every one, and name the
  assumption most worth confirming first.
- **Selection and planning only** — cluster sizing hands off to `tidb-dedicated-sizing`;
  non-MySQL sources hand off to the matching TiShift `*-to-tidb` skill. The profile YAML is
  the carrier, so the customer is never asked the same question twice.
- **Every exclusion gets a reason** — an offering marked "not recommended" without a named
  gate is incomplete output.
- **Preview and beta are always labeled** — Premium, Lake, vector search, full-text search,
  and DM on Essential/Premium are all non-GA.
- **Blockers go in the customer email**, not only in the attached report.
- **Billing is a tracked prerequisite, not a footnote** — a paid cluster cannot be created
  without a payment method (card, or an AWS/Azure/Google Cloud/Alibaba marketplace
  subscription). It lands in Next Actions with a named owner and a date before kickoff.
- **No pricing, no SLA numbers, no invented feature support** — uncertain items are marked
  "to be confirmed".

## ⚠️ Known documentation contradiction

Two TiDB docs pages disagree about vector-search tier availability:

- <https://docs.pingcap.com/tidbcloud/features/> — Starter only
- <https://docs.pingcap.com/tidbcloud/vector-search-overview/> — Starter, Essential, and
  Dedicated (beta)

This skill follows the **features page**, since that is the canonical tier-comparison
surface. The contradiction is recorded in `references/product-matrix.md`, and when it
produces a customer-facing conflict the skill's first recommended action is to confirm actual
availability with the account team — because if the broader page is right, the conflict
disappears and the architecture simplifies.

The product matrix is a snapshot taken **2026-07-29**. Re-verify before finalizing
high-stakes deals.

## Related skills

| Skill | Role |
|---|---|
| `tidb-dedicated-sizing` | Node counts and specs once the offering is chosen |
| `TiShift/*-to-tidb` | Schema conversion and migration mechanics for non-MySQL sources |
