# Handoffs

This skill does **selection and planning**. Two kinds of work belong elsewhere; hand them off
rather than approximating them.

---

## Handoff 1 — Cluster sizing → `tidb-dedicated-sizing`

**When**: the recommendation lands on **Dedicated** (or they ask for node counts,
specs, or cost shape on any tier).

**Never compute node counts, vCPU sizes, or storage in this skill.** That skill has the
official formulas from <https://docs.pingcap.com/tidbcloud/size-your-cluster/>, published
per-node QPS baselines, and storage limits per vCPU tier. Approximating them here produces
numbers that contradict the sizing skill — worse than producing none.

**Input contract** — its checklist has 10 items
(`tidb-dedicated-sizing/references/input-checklist-and-defaults.md`). Map from the intake:

| sizing input | from this skill's intake |
|---|---|
| 1. Source DB type & version | item 1 |
| 2. Total raw data size (uncompressed) | item 2 |
| 3. Peak QPS | item 3 |
| 4. Read/write ratio | item 3 |
| 5. Latency SLA (P95/P99) | item 4 |
| 6. Data growth rate & retention | not collected here — pass as unknown |
| 7. Analytical / HTAP workload and which tables | item 7 |
| 8. Cloud provider & region | item 8 |
| 9. Peak connections / concurrency | not collected here — pass as unknown |
| 10. TiKV compression ratio | not collected here — its 40% default applies |

**How to hand off**: state that sizing is the next step, list the values above as the input
block, and name the missing ones. That skill never blocks on missing inputs — it applies
documented defaults and reports them — so an incomplete handoff is still useful.

---

## Handoff 2 — Non-MySQL source → TiShift `*-to-tidb`

**When**: intake item 1 is anything other than a MySQL-compatible database.

Those skills own schema conversion, type mapping, source-specific compatibility rules, and
source-specific CDC. This skill's `feature-compatibility.md` is a first pass only — say so
rather than presenting it as a complete assessment.

**Available skills** in `TiDBTools/TiShift/`:

| Source | Skill directory |
|---|---|
| Amazon Aurora | `aurora-to-tidb` |
| CockroachDB | `cockroachdb-to-tidb` |
| Google Firestore | `firestore-to-tidb` |
| MySQL HeatWave / OCI MySQL | `heatwave-to-tidb` |
| MongoDB | `mongodb-to-tidb` |
| Neon / PostgreSQL | `neon-to-tidb` |
| OceanBase | `oceanbase-to-tidb` |
| Oracle | `oracle-to-tidb` |
| Cloud Spanner | `spanner-to-tidb` |
| SQL Server | `sqlserver-to-tidb` |
| Supabase | `supabase-to-tidb` |

If the source is not in this list (e.g. Db2, Sybase, Cassandra), say plainly that no
dedicated migration skill exists for it and that the schema/data-conversion assessment needs
manual work — do not substitute the nearest-looking skill.

**What still belongs to this skill** for a non-MySQL source: the full intake, the tier
recommendation, the feature red/yellow/green first pass, and all three deliverables. Only the
import mechanics and the deep compatibility assessment defer.

---

## Handoff order

When both apply — a non-MySQL source that lands on Dedicated — do them in this order:

1. Finish this skill's deliverables (report, profile, email).
2. Point at the TiShift skill for migration mechanics.
3. Point at `tidb-dedicated-sizing` for capacity.

The profile YAML is the carrier: it holds the intake in a form both downstream skills can
consume, so they are never asked the same question twice.
