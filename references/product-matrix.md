# TiDB Cloud Product Matrix (snapshot)

Source of truth: <https://docs.pingcap.com/tidbcloud/features/>
Snapshot taken 2026-07-29. **Re-verify before finalizing any high-stakes or large deal** —
this page changes as features graduate from preview.

Legend: ✅ generally available or public preview · 🔒 private preview (support ticket) ·
🚧 under development · ❌ not available

## The five offerings

| Offering | One-line positioning |
|---|---|
| **Starter** | Entry tier for developers and AI/SaaS applications. Pay-as-you-go, auto-scaling, SQL Editor, Data Branch. The **only** tier with vector and full-text search. |
| **Essential** | Production OLTP with autoscaling and pay-as-you-go. Adds PITR, alerting, DM (preview), backup recycle bin. |
| **Premium** (public preview) | Enterprise workloads — cross-AZ failover, Kafka changefeeds, manual backup, database audit logging, on top of the elastic model. |
| **Dedicated** | Mission-critical. Full HTAP with TiFlash, cross-AZ HA, node groups, resource control, VPC peering, CMEK, dual-region backup, GCP/Azure regions. Fixed provisioned resources, no pay-as-you-go. |
| **TiDB Cloud Lake** (public preview) | Separate product, separate doc site: <https://docs.pingcap.com/tidbcloudlake/>. Cloud-native data warehouse with compute/storage separation and independently provisioned warehouses. ANSI SQL, semi-structured data, Apache Iceberg, and native vector / full-text / geospatial engines over shared object storage. Positioned against Snowflake. |

## Feature matrix

| Category | Feature | Starter | Essential | Premium | Dedicated |
|---|---|---|---|---|---|
| **Basics** | Scalable transactional processing | ✅ | ✅ | ✅ | ✅ |
| | Analytical processing | ✅ | ✅ | ✅ | ✅ |
| | API | ✅ (preview) | ✅ (preview) | ✅ (preview) | ✅ (preview) |
| **Developer experience** | Data Branch | ✅ | ✅ | 🚧 | ❌ |
| | SQL Editor | ✅ | 🚧 | 🚧 | ❌ |
| **Resource management** | Pay-as-you-go | ✅ | ✅ | ✅ | ❌ |
| | Auto-scaling | ✅ | ✅ | ✅ | ❌ |
| | Manual cluster modification | ❌ | ❌ | ❌ | ✅ |
| | Pause & resume | ❌ | ❌ | ❌ | ✅ |
| | System maintenance window | ❌ | ❌ | 🚧 | ✅ |
| | Backup file recycle bin | ❌ | ✅ | ✅ | ✅ |
| **Specialized** | **Vector storage & search** | **✅ (preview)** | **❌** | **❌** | **❌** |
| | **Full-text search** | **✅ (preview)** | **❌** | **🚧** | **❌** |
| **Data processing** | CSV / Parquet / SQL import | ✅ | ✅ | ✅ | ✅ |
| | **Data migration from MySQL (DM)** | **❌** | **✅ (preview)** | **✅ (preview)** | **✅** |
| | CSV / Parquet / SQL export | ✅ (preview) | ✅ (preview) | 🔒 | ❌ |
| | Changefeeds to Kafka | ❌ | 🔒 | ✅ | ✅ |
| **Backup & restore** | Automatic backup | ✅ | ✅ | ✅ | ✅ |
| | Manual backup | ❌ | ❌ | ✅ | ✅ |
| | Dual-region backup | ❌ | ❌ | ❌ | ✅ |
| | Point-in-time recovery | ❌ | ✅ | ✅ | ✅ |
| | Restore | ✅ | ✅ | ✅ | ✅ |
| **Observability** | Built-in metrics | ✅ | ✅ | ✅ | ✅ |
| | Alerting | ❌ | ✅ | ✅ | ✅ |
| | SQL statement analysis | ✅ | ✅ | ✅ | ✅ |
| | Slow query log | ✅ | ✅ | ✅ | ✅ |
| | Top SQL | ❌ | ✅ (preview) | ✅ (preview) | ✅ |
| | Events | ✅ | ✅ | 🚧 | ✅ |
| | Prometheus / Grafana | ❌ | ✅ (preview) | ✅ (preview) | ✅ |
| | Datadog integration | ❌ | ✅ (preview) | ✅ (preview) | ✅ |
| | New Relic integration | ❌ | ❌ | ❌ | ✅ |
| **High availability** | Cross-AZ failover | ❌ | ❌ | ✅ | ✅ |
| | Node groups | ❌ | ❌ | ❌ | ✅ |
| | Resource control | ❌ | ❌ | 🚧 | ✅ |
| **Network** | Private endpoint | ✅ | ✅ | ✅ | ✅ |
| | Public endpoint | ✅ | ✅ | ✅ | ✅ |
| | VPC peering | ❌ | ❌ | 🔒 | ✅ |
| **Security** | Database audit logging | ❌ | 🔒 | ✅ | ✅ |
| | Console audit logging | ✅ | ✅ | ✅ | ✅ |
| | Log redaction | ✅ | ✅ | ✅ | ✅ |
| | CMEK | ❌ | ❌ | 🔒 | ✅ |
| | Dual-layer encryption | ✅ | ✅ | 🔒 | ✅ |
| | IAM | ✅ | ✅ | ✅ | ✅ |
| **Cloud regions** | AWS | ✅ | ✅ | ✅ | ✅ |
| | Alibaba Cloud | ✅ | ✅ | ✅ | ❌ |
| | Azure | ❌ | ❌ | 🚧 | ✅ |
| | Google Cloud | ❌ | ❌ | ❌ | ✅ |

## The four rows that decide most cases

Everything else is detail. These four rows produce nearly every recommendation and every
conflict:

1. **Vector search** — Starter only.
2. **Full-text search** — Starter only (Premium under development).
3. **Data migration from MySQL (DM)** — everything *except* Starter.
4. **Cloud provider** — GCP and Azure are Dedicated only.

Rows 1–2 point at Starter; rows 3–4 point away from it. That is the structural conflict the
decision tree exists to detect. See `decision-tree.md`.

## ⚠️ Known documentation contradiction — vector search

Two TiDB docs pages disagree:

| Page | Claim |
|---|---|
| <https://docs.pingcap.com/tidbcloud/features/> | Vector storage & search: Starter ✅ (preview), Essential ❌, Premium ❌, Dedicated ❌ |
| <https://docs.pingcap.com/tidbcloud/vector-search-overview/> | "available on TiDB Self-Managed, TiDB Cloud Starter, TiDB Cloud Essential, and TiDB Cloud Dedicated" (beta; v8.4.0+ required for Self-Managed and Dedicated, v8.5.0+ recommended) |

**This skill follows the features page** (Starter only) because that page is the canonical
tier-comparison surface.

**How to handle it in a customer conversation**: when vector search is a requirement and the
customer's other constraints push away from Starter, do not silently resolve the conflict.
State that the tier availability of vector search must be confirmed with the account team
before the PoC design is locked, and present it as an open question in the report. If it
turns out Essential/Dedicated do support it, the conflict disappears and the recommendation
simplifies — say so explicitly, because that is a materially better outcome for the customer.

## TiDB Cloud Lake notes

- **Public preview.** Every recommendation involving Lake must carry that caveat: feature
  availability and service limits may change.
- Capabilities per <https://docs.pingcap.com/tidbcloudlake/lake-overview/>: "ANSI SQL,
  semi-structured data processing, vector search, and AI-oriented workflows"; described as
  vector-native (embeddings, vector indexes, semantic retrieval in SQL), search-native
  (full-text indexing), and geo-native.
- Compute and storage are separated; warehouses are provisioned independently.
- Data gets in via stages / load-from-files, data-integration pipelines, and migration paths
  (e.g. from Snowflake).
- The docs do **not** clearly document the integration path between Lake and a transactional
  TiDB Cloud cluster. If a customer's design depends on that link, flag it as an open
  question rather than assuming it works.

## What not to state

- No pricing figures and no SLA percentages — those come from the account team.
- Never present a 🚧 or 🔒 item as available.
- Never present a preview/beta item as a GA commitment.
