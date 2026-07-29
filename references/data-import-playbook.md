# Data Import Playbook

Fork on three axes: **source compatibility × data volume × incremental needed**.

```
Is the source MySQL-compatible (MySQL, Aurora MySQL, RDS MySQL, Cloud SQL MySQL,
Azure MySQL Flexible Server, Alibaba RDS MySQL, MariaDB-ish)?
│
├── NO ──▶ Path 0: hand off to the matching TiShift skill. Stop here.
│
└── YES
     │
     ├── Volume ≤ 200 GB ──┬── incremental needed ──▶ Path 1: DM all-in-one
     │                     └── one-shot cutover ────▶ Path 2: Dumpling → Import
     │
     └── Volume > 200 GB ──┬── one-shot cutover ────▶ Path 3: CSV/Parquet → object store → Import
                           └── incremental needed ──▶ Path 4: Path 3 for full load
                                                              + DM incremental-only
```

---

## Path 0 — Non-MySQL source

Out of scope for this skill. Route to the matching TiShift skill (see `handoff.md`) — those
cover schema conversion, type mapping, and source-specific CDC, none of which belong here.

Still collect the intake items and still produce the tier recommendation; only the import
section defers.

---

## Path 1 — DM all-in-one (≤ 200 GB, incremental needed)

One DM job does full load and then continuous incremental replication, with no manual
handoff between phases.

**Prerequisites**: the source binlog configuration below, network connectivity below, and a
tier that supports DM (**not Starter** — see G4).

**Steps**
1. Configure source binlog (below) and restart if required.
2. Establish network connectivity (below).
3. Create the DM job in the TiDB Cloud console, selecting **existing data + incremental
   data**.
4. Monitor the full-load phase, then verify replication lag stays near zero.
5. Cut over during the maintenance window; stop writes to the source, confirm lag is zero,
   redirect the application.

**Duration**: full load dominates; incremental catch-up is minutes once lag converges.

---

## Path 2 — Dumpling → Import (≤ 200 GB, one-shot)

Export with Dumpling, import through TiDB Cloud's import feature. Simplest path — no binlog
changes on the source, no DM job to operate.

Only valid if the customer can tolerate downtime for the full export + import duration.

---

## Path 3 — Object-storage bulk load (> 200 GB, one-shot)

The primary large-volume path.

**Steps**
1. **Export** with Dumpling to CSV or Parquet.
2. **Upload** to object storage: Amazon S3, Google Cloud Storage, or Azure Blob Storage.
3. **Import** via TiDB Cloud's import feature, pointing at the bucket/prefix.

**File conventions that must be followed** (TiDB Cloud maps files to tables by name):

| Case | Pattern |
|---|---|
| Single file per table | `${db_name}.${table_name}.csv` |
| Multiple files per table | `${db_name}.${table_name}.000001.csv` — numeric suffixes may be **non-consecutive but must ascend** |
| Compressed | `${db_name}.${table_name}.csv.{gzip\|gz\|zstd\|zst\|snappy}` |

**Sizing the files**: TiDB Cloud handles very large CSVs, but performs best with **multiple
files of roughly 256 MiB**. For compressed files, target **~100 MiB** each. Split large
tables accordingly during export — this is the single biggest lever on import throughput.

**Authentication**
- Amazon S3 — **AWS Role ARN (recommended)**, or access-key credentials
- GCS — Google Cloud service account ID with the appropriate IAM permissions
- Azure Blob — SAS token; Private Link available for network-isolated access

**Common pitfalls**
- One giant file per table: no parallelism, slow import.
- Filenames not matching the pattern: tables silently unmapped.
- Suffixes out of ascending order.
- Access keys instead of Role ARN: works, but avoidable credential handling.
- Forgetting that the destination schema must exist or be creatable by the import.

---

## Path 4 — Bulk load + incremental DM (> 200 GB, incremental needed)

The standard large production migration. Combines Path 3 for the initial load with a DM job
configured for **incremental data migration only**.

**Steps**
1. Record the source binlog position / GTID before starting the export.
2. Run Path 3 for the full data set.
3. Create a DM job with **Incremental data migration** selected, starting from the recorded
   position.
4. Let it catch up; monitor until replication lag converges near zero.
5. Cut over: stop source writes, confirm zero lag, redirect the application.

**The critical detail**: capture the binlog position *before* the export begins, and ensure
the source retains binlogs covering the entire export + load window. This is what
`binlog_expire_logs_seconds` is for — see below. A too-short retention is the most common way
this path fails, and it fails *after* the expensive part is already done.

---

## Source binlog prerequisites (Paths 1 and 4)

Required on the source MySQL-compatible database:

| Variable | Required value |
|---|---|
| `log_bin` | `ON` |
| `binlog_format` | `ROW` |
| `binlog_row_image` | `FULL` |
| `binlog_expire_logs_seconds` | **≥ 86400** (1 day); **604800** (7 days) recommended |
| `binlog_transaction_compression` | `OFF` |

For Path 4 specifically, retention must exceed the full export + upload + import duration
with margin. If a 2 TB load takes three days, seven-day retention is the minimum sane setting,
not a nicety.

**Supported source versions**: self-managed MySQL 8.0 / 5.7 / 5.6; Amazon Aurora MySQL
8.0 / 5.7 / 5.6; Amazon RDS MySQL 8.0 / 5.7; Azure Database for MySQL Flexible Server
8.0 / 5.7; Google Cloud SQL for MySQL 8.0 / 5.7 / 5.6; Alibaba Cloud RDS MySQL 8.0 / 5.7.

**Migration mode**: logical mode suits data sets under 1 TiB; physical mode is recommended
above that for throughput.

---

## Network connectivity (DM)

Three options — pick one and confirm it early, because it often requires the customer's
network team and becomes the schedule bottleneck:

| Method | Available on |
|---|---|
| Public IP | All cloud providers |
| Private Link | AWS, Azure |
| VPC Peering | AWS, Google Cloud |

Note the interaction with G2: if the customer requires VPC peering, that already locked
Dedicated.

---

## Tier interactions to state explicitly

- **Starter does not support DM.** If the plan needs Path 1 or Path 4, Starter is out — say
  so in the import section too, not only in the tier rationale.
- DM is **preview** on Essential and Premium, **GA** on Dedicated. If the customer needs GA
  for a production cutover, that narrows to Dedicated.
- CSV/Parquet import is available on **all** tiers, so Path 2 and Path 3 never constrain tier
  selection.

## Always include in the import section

Prerequisites → steps → rough duration → pitfalls, plus:
- which path was chosen and **why** (quote the volume and the replication answer)
- the binlog checklist verbatim when DM is involved
- an explicit statement of the expected downtime at cutover
