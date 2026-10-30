# Feature Compatibility Flags

Classify every feature requirement they state into red, yellow, or green. Report all
three colors — a report showing only red reads as a rejection; showing only green reads as
overselling.

---

## 🔴 Red — not supported by TiDB, needs a joint replacement plan

These do not exist in TiDB. Do not soften this and do not promise a roadmap. Present the
alternative alongside the flag so they leave with a path forward rather than just a problem.

| Feature | Why it matters | Replacement approach |
|---|---|---|
| **Stored procedures** | Often carries real business logic; can be a large rewrite | Move the logic to the application layer. Scope the rewrite explicitly — count the procedures and their line count during the PoC, because this is usually the largest single migration cost. |
| **Triggers** | Silent dependencies; things break in ways nobody predicted | Application-layer logic, or CDC (changefeed) + stream processing for derived data and audit trails |
| **User-defined functions (UDF)** | Usually small but pervasive in queries | Rewrite as SQL expressions, or move into the application. Inventory every call site. |
| **Event Scheduler** | In-database cron | External scheduler: Airflow, K8s CronJob, cloud scheduler |
| **Materialized views** | Precomputed aggregates | For TP-derived reporting, evaluate TiFlash + a regular view (validate the actual queries). For independent pure AP, assess Lake via G0 rather than adding a TiKV row store solely for analytics. Otherwise assess a scheduled ETL table. |
| **XA / distributed transactions across engines** | Rare but blocking when present | Redesign around TiDB's native distributed transactions, which cover the usual motivation |
| **GIS / spatial indexes** | Geospatial querying | No spatial index in TiDB. Options: bounding-box filters on ordinary indexes, an external geo service, or TiDB Cloud Lake (geo-native, public preview). |
| **Full-text search** on a non-Starter tier | See `product-matrix.md` | Starter, Lake, or an external search service. This one is a *tier* limitation, not an engine limitation — it interacts with the decision tree (G3). |
| **Vector search** on a non-Starter tier | See `product-matrix.md` | Starter, Lake, or an external vector store. Same tier caveat — and see the documentation contradiction before treating it as settled. |

**How to phrase a red flag**: name the feature, state plainly that TiDB does not support it,
give the replacement, and give the estimation question that sizes the work ("how many
procedures, and how much logic is in them?"). Never end on the blocker alone.

---

## 🟡 Yellow — behavior differences, verify during the PoC

TiDB does these, but not identically to the source. They cause surprises after go-live rather
than errors on day one, which makes them worth listing explicitly.

| Area | What differs | What to verify in the PoC |
|---|---|---|
| **AUTO_INCREMENT** | Values are not contiguous across TiDB nodes | Any application logic assuming sequential or gapless IDs; export/report ordering that relies on ID order |
| **Transaction model** | Optimistic and pessimistic modes; retry semantics differ from MySQL's | Write-conflict handling under real concurrency; whether the application retries correctly |
| **Isolation levels** | Repeatable Read has distributed-system semantics; Serializable is not supported the same way | Any workload depending on precise isolation behavior |
| **Large transactions** | Size and duration limits apply | Batch jobs, bulk deletes/updates — usually need chunking |
| **DDL behavior** | Online DDL semantics and timing differ | Migration/deployment tooling that assumes MySQL DDL timing |
| **Character sets & collations** | Supported set differs from MySQL's full range | Sort order and comparison results for non-ASCII data — check before, not after, migration |
| **Foreign keys** | Supported in recent versions, with limitations | Cascade behavior and performance under load |
| **SQL mode / edge-case syntax** | Small divergences | Run the real query set, not a sample |

Yellow items are **PoC test cases**. Frame them that way in the report: each one becomes a
line item in the PoC validation plan.

---

## 🟢 Green — capability triggers

Requirements that TiDB answers well, and which should actively shape the recommendation.

| What they tell you | Trigger |
|---|---|
| Reporting, dashboards, aggregation over large tables, "our analytical queries are slow" | **Ask G0: is this derived from a core TP system or independent pure AP?** Slowness and query shape alone do not select TiFlash. |
| Core TP system with a few fresh, consistent reports on the same transactional data | **TiFlash direction.** Columnar replicas from TiKV for TP-derived analytics; confirm offering support (the snapshot identifies Dedicated for full HTAP). Within 50 TB is the typical profile, not a hard limit. |
| Pure AP, no TP transaction binding; especially >50 TB, large scans/joins, multi-layer modeling | **Lake direction** (public preview), including smaller pure-AP workloads. Avoid a TiKV row store with no TP purpose; see Phase 0 for the cost rationale. |
| Vector search, semantic retrieval, RAG | G3 → Starter (preview), or Lake for analytics-shaped workloads |
| Full-text search | G3 → Starter (preview), or Lake |
| Independent data warehouse, Iceberg, replacing Snowflake | G0/G7 → Lake (public preview); confirm absence of TP binding |
| Horizontal scaling, sharding pain, "we've outgrown a single instance" | TiDB's native distributed architecture — no application-level sharding |
| **Multi-tenant SaaS**, "one customer's queries slow everyone down", tenants moved onto separate instances to contain them | **Resource control** (per-user/workload quotas) and **node groups** (separate compute inside one cluster) on Dedicated. Listen for this as an operational complaint rather than a stated requirement — see G2 in `decision-tree.md`, where it is a hard lock. |
| Needs MySQL protocol compatibility | TiDB speaks the MySQL protocol; most drivers and tools work unchanged |

Slow reports are a prompt to ask about TP binding, not an automatic TiFlash recommendation.
Use item 7 and G0 to distinguish TP-derived analytics from pure AP before naming an engine.

---

## Classification rules

- **When unsure, mark "to be confirmed"** — never guess whether TiDB supports something. A
  wrong green flag is far more damaging than an open question — they may build a plan on it.
- Feature availability that depends on the **tier** (vector, full-text) must cross-reference
  the decision tree, since it may create a conflict rather than just a warning.
- Feature availability that depends on the **TiDB version** must state the version.
- If the source is not MySQL-compatible, this file is only a first pass — the matching TiShift
  skill has the authoritative per-source compatibility rules. Say that in the report rather
  than presenting this list as complete.
