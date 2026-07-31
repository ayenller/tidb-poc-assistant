# Data Masking During Migration

Most PoCs want production data, because synthetic data does not reproduce the performance
characteristics anyone is trying to measure. But production data usually carries PII, and a
PoC cluster is not a production environment. Masking is how those two facts are reconciled.

Ask about this in intake item 14. People frequently do not raise it until legal or security
review blocks the PoC — at which point the migration plan has to be redesigned, because
**masking changes the migration approach fundamentally**, not just the column values.

---

## The approach: mask in flight, in AWS DMS

**AWS DMS native Data Masking** applies transformation rules to rows while they are in
transit. The properties that matter for planning:

- **The source database is never modified.** No masked views, no `ALTER`, no type coercion,
  no shadow copy to build and keep in sync.
- **Plaintext never lands on disk or in a log.** It exists only in DMS memory.
- **Which columns get masked is declared explicitly.** Nothing is inferred from column names;
  a column called `email` is not masked because of what it is called.

Applies to MySQL-compatible sources that DMS can read. If the source is on a different cloud
(e.g. Google Cloud SQL), DMS reading cross-cloud is possible but adds latency and network
setup — flag it rather than assuming it is free.

### Two routes

| | `direct` — DMS → TiDB | `s3` — DMS → S3 (Parquet) → TiDB |
|---|---|---|
| Throughput | Row-by-row `INSERT`; slow cross-region | Bulk import; **much faster** |
| Re-running after failure | Whole task restarts | Files persist; re-run only the import |
| **Proving masking worked** | Only after it is in the database | **Before it reaches the database** — inspect the Parquet files |
| Moving parts | Fewer | One extra hop; bucket policy and lifecycle rules to get right |

**Recommend `s3` whenever masking is involved**, and say why: it gives a file-level checkpoint
where masked values can be verified *before* anything touches the target. When the whole point
is that PII must not reach the PoC cluster, "we checked afterwards" is a materially weaker
position than "we checked before loading". It also maps onto the object-storage bulk-load path
the skill already recommends above 200 GB.

---

## ⚠️ The conflict: masking is full-load only, no CDC

**This is the single most important planning consequence.** A masked DMS migration is a
**full load with no change data capture**. There is no masked equivalent of continuous
replication from a live production database.

So masking and continuous replication are **mutually exclusive**:

| They want | Consequence |
|---|---|
| Masked data **and** near-zero-downtime cutover | **Not achievable together.** Report it as a conflict. |
| Masked data, downtime acceptable | Full load with the source quiesced, or row counts taken at load start as the reference |
| Live production data, near-zero downtime | Continuous replication, unmasked — needs the PoC environment treated as production-sensitive |

**How to handle it**: this is usually not a real conflict once the purpose is separated out.
A *PoC* rarely needs both — it needs realistic data, and a cutover rehearsal is a separate
exercise. Present it that way:

1. **PoC phase** — masked full load. Realistic volume and distribution, no PII exposure. The
   source keeps running; the snapshot just goes stale, which does not matter for benchmarking.
2. **Production migration** — unmasked continuous replication, once the PoC has justified it
   and the target is under production controls.

Only if they insist on masked *and* continuously replicated is it a genuine dead end, and then
the options are: mask at the source (a masked replica they maintain, which is real work), or
accept unmasked data with compensating controls on the target. Say so plainly rather than
implying a product feature covers it.

Note the interaction with **G4**: continuous replication excludes Starter. If masking removes
the continuous-replication requirement for the PoC phase, that exclusion may no longer apply —
recheck the gate rather than carrying it forward.

---

## Type admission: which columns can actually be masked

**This is a hard constraint and the main source of unpleasant surprises.** Whether a column
can be masked depends on the internal type DMS maps it to, *not* on the MySQL type name and
not on whether it "looks like a text field".

| Class | DMS internal type | Available actions |
|---|---|---|
| **A** | `WSTRING` / `STRING` | all three, including `hash` |
| **B** | `NUMERIC` / `INT1–8` / `UINT1–8` | `digits-randomize` only — **weak** |
| **C** | `CLOB` / `NCLOB` / `BLOB` / `BYTES` / `DATE` / `DATETIME` / `REAL4` / `REAL8` / `BOOLEAN` | **none** — drop the column, or change the source type |

### Class A — fully maskable

`VARCHAR(n)`, `CHAR(n)`, **`TEXT`**, `TINYTEXT`, `ENUM`, `SET`, `TIME`.

### Class B — weak masking only

`TINYINT`, `SMALLINT`, `YEAR`, `MEDIUMINT`, `INT`, `BIGINT`, their `UNSIGNED` forms, and
`DECIMAL(p,s)`.

`digits-randomize` replaces each digit with a random digit, **preserving length and format**.
A phone number stored as `BIGINT` comes out as a different but entirely plausible phone
number. Whether that satisfies their compliance requirement is *their* call, not yours —
present it as weak masking and let them decide. Full masking requires changing the source
column to `VARCHAR`/`TEXT` first.

### Class C — cannot be masked at all

`JSON`, **`MEDIUMTEXT`**, **`LONGTEXT`**, all `BLOB` types, `BINARY`/`VARBINARY`, geometry
types, `DATE`, `DATETIME`, `TIMESTAMP`, `FLOAT`, `DOUBLE`, `BIT`.

### The three traps worth naming explicitly

1. **`TEXT` is maskable; `MEDIUMTEXT` and `LONGTEXT` are not.** Same family, opposite
   behaviour — `TEXT` maps to `WSTRING`, the larger two map to `NCLOB`. This is the easiest
   one to get wrong, and it is only caught by checking the exact type of every column.
2. **Dates cannot be masked.** Date of birth is PII in most regimes and it is class C. The
   only options are dropping the column or changing its source type.
3. **`FLOAT`/`DOUBLE` cannot be masked.** Salary and similar sensitive numerics stored as
   floats are unmaskable; as `DECIMAL` they are class B (weak only).

**What this means for the assessment**: a customer who says "mask all PII" has not yet made a
decision they can act on. The real question is per-column, and some of their sensitive columns
will land in class B or C. Ask for the **actual column types of the sensitive fields** and
flag any that cannot be fully masked as an open question with three options each: drop the
column, change the source type, or accept weak/no masking.

---

## Masking actions

| Action | Meaning |
|---|---|
| `hash` | SHA256, 64 uppercase hex characters. **Deterministic** — equal inputs produce equal outputs, so joins and unique keys survive |
| `digits-mask` | Every digit becomes a fixed character; length unchanged |
| `digits-randomize` | Every digit becomes a random digit; length and format preserved — **weak masking** |
| `drop` | Column excluded entirely; the target table has no such column |
| `keep` | Migrated as-is, in plaintext |

**`hash` being deterministic is what makes masked data usable for a PoC.** Joins still work,
unique constraints still hold, and `COUNT(DISTINCT col)` is unchanged — so query plans and
cardinality estimates behave like the real thing. Say this when a customer worries that masked
data will invalidate their benchmark. The caveat: a foreign key must be masked with the **same
action on both sides**, or the join breaks.

The mirror-image risk: the same column masked in one table and left plaintext in another. If
both hold the same data, the plaintext copies defeat the masking entirely. Worth flagging as a
whole-schema question, not a per-table one.

---

## What masking changes in the target schema

The target DDL is not a copy of the source DDL. These consequences belong in the assessment,
because several of them change application behaviour:

- **Hashed columns must be widened to `CHAR(64)`.** A source `varchar(50)` copied verbatim
  silently truncates the SHA256 — wrong data, no error.
- **`utf8mb4_0900_ai_ci` must be remapped** (TiDB does not support it; `utf8mb4_bin` is the
  usual target). **Sorting and comparison semantics change** — this needs the application
  owner's sign-off, not a silent substitution.
- **Dropped columns take their indexes with them**; composite indexes lose just that column.
- **Foreign keys are omitted during a parallel full load** — children can load before parents.
  Re-add them afterwards if the application needs them.
- **`FULLTEXT` and `SPATIAL` indexes are not migrated** (unsupported in TiDB). Cross-reference
  `feature-compatibility.md` — these are red flags in their own right.
- Nullability must be written out explicitly: `TIMESTAMP NULL DEFAULT NULL` and
  `TIMESTAMP DEFAULT NULL` differ depending on `explicit_defaults_for_timestamp`.

---

## Safety constraints to state in the assessment

| Constraint | Why it matters |
|---|---|
| **DMS engine version ≥ 3.5.4** | Below this, masking rules are **accepted and then silently ignored** — plaintext reaches the target with no error anywhere. This is the one silent failure mode in the pipeline, and the reason a version check belongs in the plan rather than the footnotes. |
| `DataMaskingErrorPolicy = STOP_TASK` | Other policies skip the offending row, which means the **unmasked original** lands on the target. |
| Log severity never above `LOGGER_SEVERITY_DEFAULT` | `DETAILED_DEBUG` writes row values into CloudWatch — plaintext PII into logs, which is usually a worse exposure than the one they were trying to prevent. |
| Source schema must not have drifted | If a column believed to be `VARCHAR` is really `BIGINT`, full masking silently degrades to weak masking. Verify types against the live instance. |
| Target tables must be empty | A full load into a non-empty table produces duplicates or primary-key conflicts. |

The 3.5.4 requirement deserves its own line in the report. A customer running an older DMS
engine would complete a migration, see no errors, and have plaintext PII in their PoC cluster.

---

## Proving it worked

Recommend three independent checks, because none is sufficient alone:

1. **Task statistics** — catches a table left suspended while the task overall reported success.
2. **Row counts** against a baseline taken before the load started.
3. **Plaintext residue scan** — catches masking that never happened, which the first two both
   report as fine.

### Exact-value verification

DMS `hash-mask` is equivalent to `UPPER(SHA2(value, 256))` over the UTF-8 bytes, so the
expected value of every masked cell is computable from the source:

```python
want = hashlib.sha256(source_value.encode("utf-8")).hexdigest().upper()
assert want == masked_value
```

This is a much stronger claim than "the format looks right" — every value is provably correct,
and on the S3 route it can be done against the Parquet files **before** anything reaches the
database. For a customer whose security team has to sign off, that distinction is the whole
point: sampling cannot prove a column is clean, but exact-value comparison over the staged
files can.

Two invariants worth checking alongside it:

- **Determinism** — two source rows holding the same value must still match after masking.
- **Cardinality** — `hash` is injective, so `COUNT(DISTINCT col)` must equal the source. A
  unique key with an unchanged distinct count proves the constraint survived.

---

## TiDB Cloud target notes

- **S3 import works on all tiers**, so masking does not by itself constrain tier selection.
- **The Role ARN must be registered in the TiDB Cloud console first** (Data → Import → Amazon
  S3 → AWS Role ARN). TiDB will not assume a role it has not been told about — a common and
  confusing first failure.
- **Starter and Essential usernames require the cluster prefix**: `<prefix>.<user>`.
- Network: Starter/Essential over the public endpoint with TLS; **Dedicated needs VPC peering
  or a private endpoint**.
- DMS output filenames do not match what TiDB resolves tables by — DMS writes
  `{prefix}/{schema}/{table}/LOAD*.parquet`, while TiDB expects `{db}.{table}[.{suffix}]`.
  The objects need renaming (a server-side copy) before import. Mention it as a real step, not
  a detail; it is a guaranteed stumble otherwise.

---

## What to put in the assessment

When masking is required, the Data Import Plan section must state:

1. **That masking forces a full load with no CDC**, and what that means for their cutover plan.
2. **The route** (`s3` recommended) and why.
3. **Per-column feasibility** for their sensitive fields — class A/B/C, with the ones that
   cannot be fully masked called out individually as open questions.
4. **The DMS ≥ 3.5.4 requirement**, as a prerequisite to verify, not a footnote.
5. **How masking will be proven**, since someone will have to sign off on it.
6. **The target-schema consequences** that change application behaviour — collation remapping
   above all.

If they have not yet decided which columns are sensitive, say that this is theirs to declare
and cannot be inferred. Nothing about the plan is knowable until that list exists, and column
types determine whether each entry on it is even achievable.
