---
status: in_progress
priority: P1
created: 2026-09-09
updated: 2026-09-09
tags: supabase, migration, data-recovery
type: bug
issue_kind: slice
context: The source local PostgreSQL dump now exists; private transfer and destination restore remain.
---

# Issue 001: Restore the original local Supabase database

## Owner direction

The owner narrowed this handoff to one database dump from `../code/site`. Do not block the database restore on the earlier multi-artifact export plan.

This is a database-only transfer. Supabase Storage metadata is in PostgreSQL, but the physical Storage object bytes are not in the SQL file and remain a separate concern if the owner needs them.

## Source export completed

On 2026-09-09, the running local `os-specs` PostgreSQL database was exported from `supabase_db_os-specs`:

```bash
docker exec supabase_db_os-specs \
  pg_dump -U postgres -d postgres --format=plain --no-owner --no-acl \
  > ~/Desktop/os.sql
```

Validated source artifact:

- path on source machine: `~/Desktop/os.sql`;
- format: complete plain PostgreSQL SQL with schema and data;
- size: `1,539,676` bytes;
- mode: `0600`;
- SHA-256: `daed4bb4b36f50289353ef9a0b12552e57ae4d491a6e06a6f96f7881f44c0cbb`;
- structural check: PostgreSQL completion marker, 95 `CREATE TABLE` statements, and 95 data `COPY` blocks.

Source comparison counts at export time:

- 99 migration records;
- 25 auth users;
- 69 organizations;
- 21 Storage metadata objects.

The owner will transfer the file privately and expects it at `~/scratch/os.sql` on the new machine before the next session. That destination file has not yet been observed or verified by this session.

## Destination restore boundary

On the new machine:

1. Verify that `~/scratch/os.sql` exists, is owner-only, is `1,539,676` bytes, and matches the recorded SHA-256. Do not print or copy its contents into logs or Git.
2. Confirm all database endpoints are local. Never use `--linked`, hosted URLs, staging credentials, or production credentials.
3. Back up the destination VM's current seeded local database before import.
4. Stop the app and other local writers. Treat `os.sql` as a full schema-and-data dump; do not apply it blindly over populated seed tables.
5. Restore only into the local Supabase PostgreSQL instance using a reviewed sequence appropriate for the destination PostgreSQL/Supabase versions.
6. Compare the four source counts above and verify the owner-selected account through the local browser workflow before accepting the restore.

The dump contains private application and authentication data. Never commit it, upload it to an unapproved service, or expose it in terminal/chat output.

## Acceptance criteria

- [x] A complete owner-only source database dump exists.
- [ ] `~/scratch/os.sql` is present and checksum-verified on the new machine.
- [ ] A pre-import backup of the destination seeded database exists.
- [ ] The database restore completes without touching a hosted project.
- [ ] Destination migration, user, organization, and Storage metadata counts match the source.
- [ ] The owner-selected account and expected data are verified through the local app.
- [ ] The source dump remains available until owner acceptance.

## Separate concern

`os.sql` contains the 21 `storage.objects` metadata rows but not their underlying object bytes. Recover the source Storage volume separately only if those files are required; do not represent metadata-only recovery as a complete Storage restore.

## Links

- `koder/docs/MIGRATION.md`
- `koder/STATE.md`
- `../code/site/supabase/config.toml`
