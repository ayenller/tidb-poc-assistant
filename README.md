# tidb-poc-assistant

A portable, agent-agnostic skill for **anyone evaluating TiDB Cloud for their own system**.

It replaces the manual loop where a vendor engineer collects your information over several
rounds, evaluates offline, and comes back with a proposal. Instead you work it out yourself in
one conversation: requirement intake → product selection → feature-compatibility warnings →
data import plan → account & billing prerequisites → deliverables.

**The user is the prospective customer, not a vendor engineer.** The skill works for the
evaluator: it says plainly when TiDB is a poor fit, leads with the features TiDB will not
support, avoids sales voice, and refers you to your TiDB contact rather than speaking for
them.

Pure Markdown — no scripts, no agent-specific frontmatter beyond `name` and `description` —
so it works with Claude Code and any agent supporting the open Agent Skills format.

## Layout

```
tidb-poc-assistant/
├── SKILL.md                          # Entry point: 6-step workflow, decision tree, hard rules
├── README.md
├── references/
│   ├── intake-checklist.md           # 13-item checklist, ask-in-rounds guidance, defaults
│   ├── decision-tree.md              # 11 ordered gates + conflict arbitration + mermaid map
│   ├── product-matrix.md             # features-page snapshot, Lake, the docs contradiction
│   ├── data-import-playbook.md       # 5 import paths, binlog prereqs (incl. managed-service
│   │                                 #   translations), CSV conventions
│   ├── feature-compatibility.md      # red / yellow / green classification + workarounds
│   ├── account-prerequisites.md      # payment method, marketplace subscription, PoC credits
│   └── handoff.md                    # input contracts for downstream skills
└── templates/
    ├── poc-report.md                 # 7-section assessment report
    ├── poc-profile.yaml              # machine-readable profile (schema v1)
    └── poc-request-email.md          # email you send to the TiDB team, with variants
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

Describe your situation in natural language:

> We run MySQL 8.0, about 800 GB, peak 30k QPS, and we need continuous sync from production
> during the migration. We're on AWS Tokyo. Which TiDB Cloud product fits, and how would we
> load the data?

Or start from nothing and let it interview you — it asks conversationally, in rounds of 3–4
questions, rather than presenting a thirteen-item form.

Output: a seven-section assessment report, a `poc-profile.yaml`, and a **PoC request email you
can send to the TiDB team** — it leads with your constraints and your conclusion so the first
reply is substantive instead of another questionnaire. All three are written to your **home
directory** by default; the skill confirms the location first and never uses a temp path or
silently overwrites an existing report.

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
Starter**. If you need both, no single offering works. The skill detects the
empty feasible set, refuses to improvise a tier, and presents three escape routes — split
clusters, TiDB Cloud Lake, or externalized search — with cost, complexity, and preview risk
for each.

## Key rules encoded in the skill

- **Never block on missing inputs** — apply documented defaults, list every one, and name the
  assumption most worth confirming first.
- **Selection and planning only** — cluster sizing hands off to `tidb-dedicated-sizing`;
  non-MySQL sources hand off to the matching TiShift `*-to-tidb` skill. The profile YAML is
  the carrier, so you are never asked the same question twice.
- **Every exclusion gets a reason** — an offering marked "not recommended" without a named
  gate is incomplete output.
- **Preview and beta are always labeled** — Premium, Lake, vector search, full-text search,
  and DM on Essential/Premium are all non-GA.
- **Blockers go in the email body**, not only in the attached report — a plan that hides them
  is worth less than one that names them.
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
produces a conflict in your plan, the skill's first recommended action is to confirm actual
availability with your TiDB contact — because if the broader page is right, the conflict
disappears and the architecture simplifies.

The product matrix is a snapshot taken **2026-07-29**. Re-verify before you commit to an
architecture.

## Related skills

| Skill | Role |
|---|---|
| `tidb-dedicated-sizing` | Node counts and specs once the offering is chosen |
| `TiShift/*-to-tidb` | Schema conversion and migration mechanics for non-MySQL sources |
