# Product support snapshot — 2026-10-30

Primary sources:
- [MySQL-compatible plan comparison](https://docs.pingcap.com/tidbcloud/features/)
- [Lake overview](https://docs.pingcap.com/tidbcloudlake/lake-overview/)
- [Filesystem introduction](https://docs.pingcap.com/tidbcloud-filesystem/filesystem-intro/)
- [Product introduction and lifecycle status](https://docs.pingcap.com/tidbcloud/tidb-cloud-intro/)

The comparison covers five **MySQL-compatible database plans**. Lake and Filesystem
are separate services. PostgreSQL-compatible Starter is excluded from this matrix;
confirm its own documentation when requested. A PostgreSQL source does not by itself
mean a PostgreSQL-compatible target.

## MySQL-compatible plan matrix

Legend follows the source: ✅ means GA **or public preview**, not necessarily GA;
🔒 requires private-preview approval; 🚧 is under development; ❌ unavailable.
Explicit public-preview annotations are retained below. Check exact regions and current
feature status before a commitment. Never silently admit 🔒 or 🚧 as supported.

| Capability | Starter | Essential | Premium | BYOC | Dedicated |
|---|---|---|---|---|---|
| Transaction processing | ✅ | ✅ | ✅ | ✅ | ✅ |
| Analytical processing | ✅ | ✅ | ✅ | ✅ | ✅ |
| API | ✅ preview | ✅ preview | ✅ preview | 🚧 | ✅ preview |
| Data Branch | ✅ | ✅ | 🚧 | ❌ | ❌ |
| SQL Editor | ✅ | 🚧 | 🚧 | ❌ | ❌ |
| Pay as you use | ✅ | ✅ | ✅ | ❌ | ❌ |
| Automatic scaling | ✅ | ✅ | ✅ | ✅ | ❌ |
| Manual modification | ❌ | ❌ | ❌ | ✅ | ✅ |
| Password setting | ✅ | ✅ | ✅ | ✅ | ✅ |
| Pause / resume | ❌ | ❌ | ❌ | ✅ | ✅ |
| Maintenance window | ❌ | ❌ | 🚧 | 🚧 | ✅ |
| Backup recycle bin | ❌ | ✅ | ✅ | ✅ | ✅ |
| Vector search | ✅ preview | ❌ | ❌ | ❌ | ❌ |
| Full-text search | ✅ preview | ❌ | 🚧 | 🚧 | ❌ |
| CSV / Parquet / SQL import | ✅ | ✅ | ✅ | ✅ | ✅ |
| MySQL data migration (DM) | ❌ | ✅ preview | ✅ preview | ✅ preview | ✅ |
| File export to local / object storage | ✅ preview | ✅ preview | ✅ preview | ✅ preview | ❌ |
| Changefeeds to Kafka / MySQL-compatible targets | ❌ | 🔒 | ✅ | ✅ | ✅ |
| Automatic backup | ✅ | ✅ | ✅ | ✅ | ✅ |
| Manual backup | ❌ | ❌ | ✅ | ✅ | ✅ |
| Dual-region backup | ❌ | ❌ | 🚧 | 🚧 | ✅ |
| PITR | ❌ | ✅ | ✅ | ✅ | ✅ |
| Restore | ✅ | ✅ | ✅ | ✅ | ✅ |
| Built-in metrics | ✅ | ✅ | ✅ | ✅ | ✅ |
| Alerting | ❌ | ✅ | ✅ | ✅ | ✅ |
| SQL analysis | ✅ | ✅ | ✅ | ✅ | ✅ |
| Slow queries | ✅ | ✅ | ✅ | ✅ | ✅ |
| Top SQL / Top RU | ❌ | ✅ Top RU preview | ✅ Top RU preview | ✅ Top RU preview | ✅ |
| Events | ✅ | ✅ | ✅ | ✅ | ✅ |
| Prometheus / Grafana | ❌ | ✅ preview | ✅ preview | 🔒 | ✅ |
| Datadog | ❌ | ✅ preview | ✅ preview | ❌ | ✅ |
| New Relic | ❌ | ✅ preview | ✅ preview | ❌ | ✅ |
| Cross-AZ failover | ❌ | ❌ | ✅ | ✅ | ✅ |
| Single-AZ deployment | ❌ | ✅ | ❌ | ✅ | ❌ |
| Node groups | ❌ | ❌ | ❌ | ❌ | ✅ |
| Resource control | ❌ | ❌ | 🚧 | 🚧 | ✅ |
| Private endpoint | ✅ | ✅ | ✅ | ✅ | ✅ |
| Public endpoint | ✅ | ✅ | ✅ | ✅ | ✅ |
| VPC peering | ❌ | ❌ | 🔒 | 🔒 | ✅ |
| Database audit | ❌ | 🔒 | ✅ | ✅ | ✅ preview |
| Console audit | ✅ | ✅ | ✅ | ✅ | ✅ |
| Log redaction | ✅ | ✅ | ✅ | ✅ | ✅ |
| CMEK | ❌ | ❌ | 🔒 | 🚧 | ✅ |
| Dual-layer encryption | ✅ | ✅ | 🔒 | 🚧 | ✅ |
| IAM / SSO | ✅ | ✅ | ✅ | ✅ | ✅ |
| AWS | ✅ | ✅ | ✅ | ✅ | ✅ |
| Alibaba Cloud | ✅ | ✅ | ✅ | 🚧 | ❌ |
| Azure | ❌ | ❌ | 🚧 | 🚧 | ✅ |
| Google Cloud | ❌ | ❌ | ❌ | ❌ | ✅ |

The introduction identifies Essential, Premium, and BYOC as public preview. Starter's
MySQL offering is GA on AWS and preview on Alibaba; Dedicated is GA on AWS/Azure/GCP.
Feature-level preview labels still apply within GA plans.

## Billing interpretation

The feature row "pay as you use" does not establish the billing unit. The
[billing page](https://docs.pingcap.com/tidbcloud/tidb-cloud-billing/), verified 2026-10-30,
says Essential is billed on provisioned RCUs, Premium on consumed RCUs and used storage,
and BYOC has cloud infrastructure, provisioned-vCPU service, and support charges.
Do not turn the matrix into a price quote or assume automatic scaling means usage billing.

## BYOC selection boundary

BYOC puts the data plane in your own AWS account while PingCAP manages the control plane
and operations. Select it for an explicit customer-cloud infrastructure requirement,
not merely access to a customer bucket. AWS support does not mean every AWS region.
[Onboarding](https://docs.pingcap.com/tidbcloud/byoc-onboarding-overview/) requires work
with the TiDB Cloud team; record cloud preparation and permissions as prerequisites,
not an ordinary self-service database launch.

## Lake and Filesystem

Use [lake-and-filesystem.md](lake-and-filesystem.md) for service fit and PoC questions.
Do not apply the MySQL plan matrix to either service. "Analytical processing ✅" does
not establish TiFlash support; confirm the required engine for the chosen plan.

## Conflicting or incomplete documentation

The previous snapshot recorded a broader claim at `vector-search-overview/` than the
comparison table. That overview could not be read during this refresh, so its old claim
is not verified current evidence. Use the current matrix for initial filtering; if vector
support changes the architecture, confirm plan/version availability before final selection.
Record a discrepancy only with its verification date and source; do not call an old conflict
confirmed today. ✅ without a lifecycle annotation does not resolve GA status.

No price quotes, SLA percentages, or unsupported feature promises.
