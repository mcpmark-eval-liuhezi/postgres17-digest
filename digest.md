# PostgreSQL 17 Release Digest

**For application engineers migrating from PostgreSQL 16.** Verified against the PostgreSQL project's own release announcement and independent sources (full list at the bottom).

## Release date

**PostgreSQL 17 was released on September 26, 2024** by the PostgreSQL Global Development Group.

## Headline changes that matter to application engineers

- **SQL/JSON `JSON_TABLE`** — JSON columns can finally be shredded into ordinary rows with real SQL. `JSON_TABLE()` can be used in the `FROM` clause of a `SELECT` as a tuple source, turning nested JSON into columns you can filter, join and aggregate. This is the flagship developer feature of the release; alongside it come the SQL/JSON constructors (`JSON()`, `JSON_SCALAR()`, `JSON_SERIALIZE()`) and query functions (`JSON_EXISTS()`, `JSON_QUERY()`, `JSON_VALUE()`), plus more `jsonpath` expressions with conversions to native numeric, boolean, string and date/time types.
- **`MERGE` gained `RETURNING`** — a `RETURNING` clause can now be appended to `MERGE`, returning values based on each row inserted, updated or deleted. The `merge_action()` function reports whether each returned row was an `INSERT`, `UPDATE` or `DELETE`. **See the correction below** on what it can and cannot return.
- **Logical replication survives failover and major-version upgrades** — replication slots can now be marked for failover (`failover` property, `sync_replication_slots`, and failover control on `CREATE`/`ALTER SUBSCRIPTION`), so a logical replica keeps working after a standby is promoted. This only applies to clusters already on 17 or later. The new `pg_createsubscriber` tool converts a physical standby into a logical replica, and `pg_upgrade` now preserves logical slots on publishers and full subscription state on subscribers.
- **Incremental backups are built in** — `pg_basebackup` now supports incremental backups, and the new `pg_combinebackup` utility reconstructs a full backup from a base plus incrementals. Incrementals copy only the blocks that changed since the referenced backup, driven by the new WAL summarizer process, and are much faster to restore than replaying WAL.
- **`VACUUM` uses up to 20x less memory** — the new internal memory structure for dead-tuple storage (based on adaptive radix trees / the new TID store) is dramatically more memory-efficient than the old fixed array, which improves vacuum speed and frees shared resources for your workload. The old hard 1GB cap on tracked dead tuples per index-vacuum pass is gone, which often removes the need for multiple index vacuum phases. This is genuinely in the release — the "up to" is doing work, so real gains depend on your data.
- **Bulk loading and exporting got faster** — up to **2x faster `COPY` when exporting large rows**, further `COPY` improvements when source and destination encodings match, and a new `ON_ERROR ignore` option that lets a `COPY` import continue past bad rows.
- **Throughput and query execution** — high-concurrency workloads can see **up to 2x better write throughput** from WAL processing improvements; the new streaming I/O interface speeds up sequential scans and `ANALYZE`; B-tree index lookups on `IN` clauses are faster; BRIN indexes can now be built in parallel.
- **Other developer-facing niceties** — identity columns and exclusion constraints on partitioned tables, `MERGE` can modify updatable views with a new `WHEN NOT MATCHED BY SOURCE` clause, `postgres_fdw` can push down `EXISTS`/`IN` subqueries, `EXPLAIN` shows local I/O read/write times with new `SERIALIZE` and `MEMORY` options, and there's a new built-in immutable collation provider plus a `pg_maintain` predefined role.

## Migration note

A dump/restore (`pg_dumpall`), `pg_upgrade`, or logical replication is required to move data to version 17. Version 17 also contains several changes that can affect compatibility with previous releases (e.g. maintenance operations now use a safe `search_path`, and some `pg_stat_progress_vacuum` / `pg_stat_slru` column names were renamed) — review the release notes before upgrading.

## Corrections to the original working notes

The raw notes contained one date error and one overstated capability. Everything else checked out.

1. **Release date — WRONG.** The notes had it as "August 2024, or did someone say September 18?" Neither is right: PostgreSQL 17 was released on **September 26, 2024**.
2. **`MERGE ... RETURNING` can return both old and new values — WRONG (that's PostgreSQL 18).** In PostgreSQL 17, `MERGE ... RETURNING` works and returns the *new* values of the target row for `INSERT`/`UPDATE` actions and the *old* values for `DELETE` actions. It **cannot** return both old and new values in one go. The `OLD` and `NEW` aliases that make that possible were introduced in PostgreSQL 18 (commit `80feb727c8`). For now, `merge_action()` is the way to tell which action produced each row.
3. **JSON_TABLE, logical replication failover slots, incremental backups, vacuum memory — CORRECT as written.** Details above, including the caveats: failover slots only help clusters already on 17+, and the vacuum memory figure is "up to" 20x.

## Sources

- PostgreSQL 17 Released! — official release announcement: https://www.postgresql.org/about/news/postgresql-17-released-2936
- PostgreSQL 17.0 Release Notes — official documentation: https://www.postgresql.org/docs/release/17.0
- MERGE — PostgreSQL 17 official documentation: https://www.postgresql.org/docs/17/sql-merge.html
- Waiting for PostgreSQL 17 — Add RETURNING support to MERGE (Depesz): https://www.depesz.com/2024/03/19/waiting-for-postgresql-17-add-returning-support-to-merge
- PostgreSQL 18 RETURNING & MERGE RETURNING Explained (pgEdge): https://www.pgedge.com/blog/postgresql-18-returning-enhancements-a-game-changer-for-modern-applications
- Waiting for Postgres 17: Incremental base backups (pganalyze): https://pganalyze.com/blog/5mins-postgres-17-incremental-backups
- Waiting for Postgres 17: Faster VACUUM with Adaptive Radix Trees (pganalyze): https://pganalyze.com/blog/5mins-postgres-17-faster-vacuum-adaptive-radix-trees
