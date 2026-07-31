# Data Import Playbook

**First, check whether masking is required** (intake item 14). If production data must be
masked before it reaches the PoC cluster, the fork below does not apply — go to
`data-masking.md` and use **Path 5**. Masking is a full-load-only approach, so it overrides
the incremental axis entirely rather than adding to it.

Otherwise, fork on three axes: **source compatibility × data volume × incremental needed**.

```
Must the data be masked before it reaches the PoC cluster?
│
├── YES ──▶ Path 5: AWS DMS native masking → S3 → TiDB.
│           Full load only — no CDC. Overrides everything below.
│           ⚠️ + near-zero-downtime cutover = CONFLICT (see data-masking.md)
│
└── NO
     │
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
          └── Volume > 200 GB ──┬── one-shot cutover ────▶ Path 3: CSV/Parquet → object store
                                └── incremental needed ──▶ Path 4: Path 3 for full load
                                                                   + DM incremental-only
```

**Do not confuse the two "DM"s.** *DM* here means TiDB Cloud **Data Migration** (Paths 1 and
4). *DMS* means **AWS Database Migration Service**, which is what performs masking in Path 5.
They are different products from different vendors, and mixing them up in a customer-facing
document is both confusing and embarrassing.

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

Only valid if they can tolerate downtime for the full export + import duration.

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
the source retains binlogs covering the entire export + load window. Retention is what
governs this — see below, including the **managed-service section**, because Cloud SQL, RDS,
and Azure each configure it differently and `binlog_expire_logs_seconds` is not always the
lever. A too-short retention is the most common way this path fails, and it fails *after* the
expensive part is already done.

---

## Path 5 — Masked full load (masking required)

When PII must be masked before it reaches the PoC cluster. Full detail in `data-masking.md`;
the planning shape is:

```
Source ──▶ AWS DMS ──▶ S3 (Parquet) ──▶ TiDB
            masking                     IMPORT INTO
         in flight; source
          never modified
```

**Recommend the S3 variant over DMS writing directly into TiDB**, because it gives a
checkpoint where masked values can be verified *before* anything reaches the target — which
is the entire point when the requirement is that PII must not land there.

**What makes this path different from Paths 1–4:**

- **Full load only. No CDC, no incremental.** There is no masked equivalent of continuous
  replication. If they also need near-zero-downtime cutover, that is a **conflict** — see the
  conflict section in `data-masking.md`, which usually resolves by separating the PoC phase
  (masked, stale snapshot, fine for benchmarking) from the production migration (unmasked,
  replicated, under production controls).
- **No binlog prerequisites.** A full-load task needs only `SELECT` on the source, so the
  entire binlog checklist below — and the managed-service translation problem with it —
  simply does not apply. For a source where binlog access is difficult or unavailable, this
  is a genuine advantage worth naming.
- **Not every column can be masked.** Type admission (class A/B/C) decides it, and `JSON`,
  `MEDIUMTEXT`/`LONGTEXT`, dates, and floats cannot be masked at all. Establish the sensitive
  column list *and their exact types* before promising anything.
- **The target DDL changes** — hashed columns widen to `CHAR(64)`, collations get remapped,
  foreign keys and `FULLTEXT`/`SPATIAL` indexes are omitted.
- **DMS engine must be ≥ 3.5.4**, or masking rules are accepted and silently ignored.

**Duration**: comparable to Path 3 — the bulk load dominates. Masking itself is not the
bottleneck.

**Tier impact**: none directly. S3 import works on all tiers. But if masking removes the
continuous-replication requirement for the PoC phase, **recheck G4** — the exclusion of
Starter may no longer apply.

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

## ⚠️ Managed services do not expose these variables the same way

The table above is written for **self-managed MySQL**. On a managed service, several of those
variables are provider-controlled, renamed, or set through a completely different mechanism —
`SET GLOBAL binlog_expire_logs_seconds = 604800` will simply fail or be ignored.

**Never present the self-managed checklist as if it applies directly to a managed instance.**
Translate it, and ask them to verify against their live instance before committing to a
schedule. Getting retention wrong is only discovered after the expensive part is done.

Most sources are managed services, so this is the common case, not the exception.

### Google Cloud SQL for MySQL

| Requirement | How it actually works |
|---|---|
| Enable binary logging | `gcloud sql instances patch INSTANCE_NAME --enable-bin-log`. **Enabling it on an existing instance restarts the instance** — schedule that. PITR is on by default for Enterprise Plus; Enterprise editions need it enabled explicitly. |
| Retention | **Not** `binlog_expire_logs_seconds`. Use `--retained-transaction-log-days`. **Cloud SQL Enterprise: 1–7 days (default 7). Cloud SQL Enterprise Plus: 1–35 days (default 14).** Ask which edition — the ceiling differs by 5×, and it decides how much margin a large load has. |
| Trade-off to mention | More retained days requires more storage. |

### Amazon RDS for MySQL / Aurora MySQL

| Requirement | How it actually works |
|---|---|
| Enable binary logging | Tied to **automated backups**. Setting the backup retention period to zero disables binary logging (turns off `log_bin`). So binlog availability depends on a backup setting, not a binlog setting. |
| `binlog_format` | Set via the **DB parameter group**, not `SET GLOBAL`. On RDS for MySQL it is dynamic — no instance reboot needed. |
| Retention | `CALL mysql.rds_set_configuration('binlog retention hours', N);` — hours, not seconds. **Default is `NULL`, which means binlogs are not retained at all.** Maximum is **168 hours (7 days)** for MySQL DB instances; `0` is not permitted. |
| The trap | The `NULL` default means a customer who has "backups enabled" may still be retaining no binlogs. Have them run `CALL mysql.rds_show_configuration;` and read the actual value rather than assuming. |

### Azure Database for MySQL Flexible Server

| Requirement | How it actually works |
|---|---|
| `binlog_expire_logs_seconds` | Settable as a **server parameter** through the portal/CLI. Microsoft's own guidance for outbound replication suggests starting at **at least two days**. |
| `binlog_row_image` | Set to `FULL` via server parameters. |
| Extra gotcha | `lower_case_table_names` must be **consistent between source and target**. It defaults to `1` on Flexible Server, which differs from a typical Linux self-managed MySQL default of `0`. A mismatch causes table-name resolution failures that look like missing tables. |

### Alibaba Cloud RDS MySQL

Configured through the console's parameter settings rather than `SET GLOBAL`. Confirm the
current values with them directly — treat the specifics as **to be confirmed** rather
than assuming they match either the self-managed defaults or another provider's model.

### How to present this

1. Give them the five-row requirements table — it states the *goal*.
2. Give them the provider-specific translation above — it states *how*.
3. Ask them to **read back the live values** from their instance, not from memory or from the
   provider's defaults page.
4. Only then commit to a cutover date.

---

## Network connectivity (DM)

Three options — pick one and confirm it early, because it often requires their
network team and becomes the schedule bottleneck:

| Method | Available on |
|---|---|
| Public IP | All cloud providers |
| Private Link | AWS, Azure |
| VPC Peering | AWS, Google Cloud |

Note the interaction with G2: if they require VPC peering, that already locked
Dedicated.

---

## Tier interactions to state explicitly

- **Starter does not support DM.** If the plan needs Path 1 or Path 4, Starter is out — say
  so in the import section too, not only in the tier rationale.
- DM is **preview** on Essential and Premium, **GA** on Dedicated. If they need GA
  for a production cutover, that narrows to Dedicated.
- CSV/Parquet import is available on **all** tiers, so Path 2 and Path 3 never constrain tier
  selection.

## Always include in the import section

Prerequisites → steps → rough duration → pitfalls, plus:
- which path was chosen and **why** (quote the volume and the replication answer)
- the binlog checklist when DM is involved — the five-row requirements table **plus** the
  provider-specific translation if the source is a managed service, never the raw table alone
- when masking is required: that it forces full-load-only, the per-column feasibility of their
  sensitive fields, the DMS ≥ 3.5.4 prerequisite, and how masking will be proven
- an explicit statement of the expected downtime at cutover
