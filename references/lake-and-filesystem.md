# Lake and Filesystem — service-specific planning

Verified 2026-10-30. These services have separate documentation and public-preview status.

## Lake

Source: [Lake overview](https://docs.pingcap.com/tidbcloudlake/lake-overview/).

Lake provides managed warehouse compute separately from object storage. Its overview
covers SQL analytics plus semi-structured, vector, text-search, and geospatial workloads.
For independent AP, assess Lake; for analytics derived from a core TP system, retain the
TiFlash direction in `decision-tree.md`. Preserve the earlier 50 TB guidance as a selection
signal, not a service limit.

Collect representative SQL, data types, freshness, concurrency, scan/join patterns, retrieval
needs, and acceptance criteria. Confirm exact source connectors, ingestion methods, regions,
SQL compatibility, security, and recovery through the relevant Lake guides before claiming
support. The overview alone does not verify a specific transactional-TiDB integration,
Iceberg workflow, or migration mechanism. Do not apply MySQL feature red flags, DM paths,
or database-plan billing rules to Lake without checking its documentation.

## Filesystem

Source: [Filesystem introduction](https://docs.pingcap.com/tidbcloud-filesystem/filesystem-intro/).

Choose this service for files that must persist beyond machines or agent sessions, or be
shared across environments. It does not require provisioning a TiDB database. Clients can
use `ti fs` without mounting, or mount a directory for tools that need local paths. Owner
and scoped tokens control access. Layers isolate edits; checkpoints and selected commits
support reviewing changes before applying them to the base.

Ask only the relevant questions:
- Which files, sizes/counts, read/write patterns, and persistence period?
- Which users, CI jobs, agents, and machines share them? Which paths and operations should
  each token permit? Record scopes, never token values.
- Is a mount necessary, on which OS/runtime, or is CLI/SDK access sufficient?
- Are isolated edits, checkpoints, and commits to the shared base needed?
- Which region, usage limits, billing arrangement, and preview tolerance are required?

Consult the introduction's region/limitation, mount, authorization, SDK, and billing links
before committing to platform support, semantics, concurrency behavior, limits, or cost.
Do not promise full POSIX compatibility or treat this as Lake storage, S3, a SQL database,
or a database replication mechanism.

A Filesystem-only PoC tests persistence across sessions, cross-environment access,
permitted/denied token operations, required mount behavior, and layer/checkpoint handling
when relevant. Mark database QPS, binlogs, SQL conversion, cutover, and database sizing
not applicable. If files accompany a database/warehouse PoC, plan each service separately;
do not assert an automatic integration or create a database just to use Filesystem.
