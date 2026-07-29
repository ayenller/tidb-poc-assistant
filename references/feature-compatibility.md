# Feature Compatibility Flags

Classify every feature requirement the customer states into red, yellow, or green. Report all
three colors — a report showing only red reads as a rejection; showing only green reads as
overselling.

---

## 🔴 Red — not supported by TiDB, needs a joint replacement plan

These do not exist in TiDB. Do not soften this and do not promise a roadmap. Present the
alternative alongside the flag so the conversation stays constructive.

| Feature | Why it matters | Replacement approach |
|---|---|---|
| **Stored procedures** | Often carries real business logic; can be a large rewrite | Move the logic to the application layer. Scope the rewrite explicitly — count the procedures and their line count during the PoC, because this is usually the largest single migration cost. |
| **Triggers** | Silent dependencies; things break in ways nobody predicted | Application-layer logic, or CDC (changefeed) + stream processing for derived data and audit trails |
| **User-defined functions (UDF)** | Usually small but pervasive in queries | Rewrite as SQL expressions, or move into the application. Inventory every call site. |
| **Event Scheduler** | In-database cron | External scheduler: Airflow, K8s CronJob, cloud scheduler |
| **Materialized views** | Precomputed aggregates | TiFlash + a regular view often removes the need entirely (real-time columnar aggregation instead of a refresh cycle). Otherwise a scheduled ETL table. |
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

| Customer statement | Trigger |
|---|---|
| Reporting, dashboards, aggregation over large tables, "our analytical queries are slow" | **Recommend TiFlash columnar replicas.** Real-time columnar replication from TiKV — no ETL, no separate warehouse, queries route automatically. Note that full HTAP capability lives in Dedicated/Premium. |
| Mixed transactional + analytical on the same data (HTAP) | TiFlash, same as above. This is TiDB's core differentiator — say so. |
| Vector search, semantic retrieval, RAG | G3 → Starter (preview), or Lake for analytics-shaped workloads |
| Full-text search | G3 → Starter (preview), or Lake |
| Heavy OLAP, data warehouse, Iceberg, replacing Snowflake | G7 → Lake (public preview) |
| Horizontal scaling, sharding pain, "we've outgrown a single instance" | TiDB's native distributed architecture — no application-level sharding |
| Needs MySQL protocol compatibility | TiDB speaks the MySQL protocol; most drivers and tools work unchanged |

TiFlash is the most commonly missed green flag. Customers describe slow reports without
recognizing it as a product-selection input — listen for it in item 7 of the intake.

---

## Classification rules

- **When unsure, mark "to be confirmed"** — never guess whether TiDB supports something. A
  wrong green flag in a pre-sales document is far more damaging than an open question.
- Feature availability that depends on the **tier** (vector, full-text) must cross-reference
  the decision tree, since it may create a conflict rather than just a warning.
- Feature availability that depends on the **TiDB version** must state the version.
- If the source is not MySQL-compatible, this file is only a first pass — the matching TiShift
  skill has the authoritative per-source compatibility rules. Say that in the report rather
  than presenting this list as complete.
