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
│   ├── intake-checklist.md           # 14-item checklist, ask-in-rounds guidance, defaults
│   ├── decision-tree.md              # family routing, requirement gates, conflicts, worked cases
│   ├── product-matrix.md             # dated MySQL plan matrix, BYOC, lifecycle evidence
│   ├── lake-and-filesystem.md        # service fit, focused intake, validation boundaries
│   ├── data-import-playbook.md       # 6 import paths, binlog prereqs (incl. managed-service
│   │                                 #   translations), CSV conventions
│   ├── feature-compatibility.md      # red / yellow / green classification + workarounds
│   ├── data-masking.md               # DMS native masking: type admission, the no-CDC
│   │                                 #   conflict, verification, target DDL effects
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
questions, rather than presenting a fourteen-item form.

Output: a seven-section assessment report, a `poc-profile.yaml`, and a **PoC request email you
can send to the TiDB team** — it leads with your constraints and your conclusion so the first
reply is substantive instead of another questionnaire. All three are written to your **home
directory** by default; the skill confirms the location first and never uses a temp path or
silently overwrites an existing report.

## What it evaluates

- **MySQL-compatible database plans**: Starter, Essential, Premium, BYOC, Dedicated.
- **Lake**: independent analytical workloads; TP-derived reporting keeps the TiFlash
  direction on a confirmed supporting database plan.
- **Filesystem**: persistent/shared files across machines, CI jobs, and AI-agent sessions,
  without requiring a database. Includes access scopes, mounts, layers/checkpoints, and
  service-specific PoC criteria.
- **PostgreSQL-compatible Starter**: identified as a separate public-preview track; the
  MySQL matrix is not reused as evidence for it.

The workflow routes by family first, then intersects region, ownership, features, recovery,
and integration requirements. Mixed workloads get a recommendation and validation plan
per service. BYOC is selected for own-cloud data-plane requirements, not merely a customer
bucket. Missing answers remain documented assumptions or open questions.

The verified 2026-10-30 snapshot comes from the official
[feature comparison](https://docs.pingcap.com/tidbcloud/features/),
[Lake overview](https://docs.pingcap.com/tidbcloudlake/lake-overview/), and
[Filesystem introduction](https://docs.pingcap.com/tidbcloud-filesystem/filesystem-intro/).
[Product introduction](https://docs.pingcap.com/tidbcloud/tidb-cloud-intro/) provides
plan lifecycle status. Public/private preview and under-development support are kept distinct;
a checkmark alone is not a GA promise. Re-verify before committing to an architecture.

The earlier [TiFlash vs Lake selection guidance](https://docs.google.com/document/d/1ZDQs9fBaLbnURM2aTshQoowIZPVU72j2Q2scXlzEw-U/edit?tab=t.0)
is retained: TP-derived analytics → TiFlash; independent pure AP → Lake. The 50 TB marker
is a selection signal, not a capacity limit or Lake minimum.

## Planning and outputs

- Flag target-specific compatibility gaps, behavior differences, and requirement conflicts.
- Plan database loading, replication, masking, and cutover, or service-specific file/data
  access. Masking limitations remain specific to the documented DMS route.
- Track applicable billing/access readiness; do not apply database credit-recycling rules
  to Lake or Filesystem without verifying their own terms.
- Generate an assessment report, YAML profile, and request email, including owners,
  assumptions, unresolved checks, preview risks, and specialist handoffs.

Selection and planning only: no infrastructure deployment, cluster capacity math, price
quotes, SLA commitments, or deep schema conversion. Worked cases in
`references/decision-tree.md` cover family routing, ownership conflicts, current New Relic
support, private-preview access, and Filesystem-only requests.

Historical vector-doc discrepancies are dated context, not a newly verified contradiction.
If search availability changes the architecture, confirm the exact plan/version first.

## Related skills

| Skill | Role |
|---|---|
| `tidb-dedicated-sizing` | Node counts and specs once the offering is chosen |
| `TiShift/*-to-tidb` | Schema conversion and migration mechanics for non-MySQL sources |
