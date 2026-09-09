---
status: blocked
priority: P1
created: 2026-09-09
updated: 2026-09-09
tags: supabase, migration, data-recovery
type: bug
issue_kind: slice
context: The VM received repository files and seed SQL, but not the old machine's local Supabase database or Storage volume.
---

# Issue 001: Restore the original local Supabase data

## Problem

The VM database is not a 1:1 copy of the old machine. `os.zip` intentionally excluded Docker data, and no PostgreSQL dump or Storage backup exists in the transfer archive or `~/scrap/`. The VM was initialized with `supabase db reset --local`, so its data comes from repository migrations and six seed files.

Observed VM baseline on 2026-09-09:

- 99 migration records;
- 25 local auth users;
- 60 organizations;
- the reported founder account and its content are seed-generated;
- 12 local Supabase containers are healthy, but they contain only the rebuilt seed state.

The old machine's preserved Supabase volume is therefore the required source. Do not delete, reset, prune, or recreate that stack.

## Source-machine export handoff

Run from the old OneSource workspace with the app stopped so no writes occur during export. Starting the existing local Supabase stack is acceptable; **never run `supabase db reset` on the source**.

1. Open the harness from `root/`, read the root and `code/site` handoffs, and verify the old local stack and volume still exist.
2. Confirm every database endpoint is local (`127.0.0.1`/`localhost`), not linked staging or production.
3. Create a private export directory outside Git:

   ```bash
   export EXPORT_DIR="$HOME/scrap/onesource-supabase-export-$(date +%Y%m%dT%H%M%S)"
   mkdir -m 700 -p "$EXPORT_DIR"
   cd ../code/site
   ```

4. Produce the portable Supabase SQL set using the CLI version available on the old machine:

   ```bash
   supabase db dump --local --role-only --file "$EXPORT_DIR/roles.sql"
   supabase db dump --local --file "$EXPORT_DIR/schema.sql"
   supabase db dump --local --data-only --use-copy --file "$EXPORT_DIR/data.sql"
   ```

5. Also retain a full custom-format PostgreSQL fallback. Discover the existing database container rather than assuming its name:

   ```bash
   DB_CONTAINER=$(docker ps --format '{{.Names}}' | awk '/^supabase_db_/ {print; exit}')
   test -n "$DB_CONTAINER"
   docker exec "$DB_CONTAINER" \
     pg_dump -U postgres -d postgres -Fc --no-owner --no-acl \
     > "$EXPORT_DIR/postgres-full.dump"
   ```

6. Export local Storage object bytes as well as database metadata:

   ```bash
   STORAGE_CONTAINER=$(docker ps --format '{{.Names}}' | awk '/^supabase_storage_/ {print; exit}')
   test -n "$STORAGE_CONTAINER"
   mkdir -m 700 "$EXPORT_DIR/storage"
   docker cp "$STORAGE_CONTAINER:/mnt/." "$EXPORT_DIR/storage/"
   tar -C "$EXPORT_DIR" -czf "$EXPORT_DIR/storage.tar.gz" storage
   rm -rf "$EXPORT_DIR/storage"
   ```

7. Record non-secret compatibility and comparison evidence:

   ```bash
   {
     date --iso-8601=seconds
     uname -m
     docker --version
     supabase --version
     docker exec "$DB_CONTAINER" postgres --version
   } > "$EXPORT_DIR/versions.txt"

   docker exec -i "$DB_CONTAINER" psql -U postgres -d postgres -At \
     -v ON_ERROR_STOP=1 > "$EXPORT_DIR/counts.txt" <<'SQL'
   select 'migrations=' || count(*) from supabase_migrations.schema_migrations;
   select 'users=' || count(*) from auth.users;
   select 'organizations=' || count(*) from public.organizations;
   select 'storage_objects=' || count(*) from storage.objects;
   SQL
   ```

8. Verify outputs are non-empty, make the bundle owner-only, archive it, and write a checksum:

   ```bash
   test -s "$EXPORT_DIR/schema.sql"
   test -s "$EXPORT_DIR/data.sql"
   test -s "$EXPORT_DIR/postgres-full.dump"
   chmod -R go-rwx "$EXPORT_DIR"
   EXPORT_PARENT=$(dirname "$EXPORT_DIR")
   EXPORT_NAME=$(basename "$EXPORT_DIR")
   tar -C "$EXPORT_PARENT" -czf "$EXPORT_DIR.tar.gz" "$EXPORT_NAME"
   chmod 600 "$EXPORT_DIR.tar.gz"
   (
     cd "$EXPORT_PARENT"
     sha256sum "$EXPORT_NAME.tar.gz" > "$EXPORT_NAME.tar.gz.sha256"
   )
   chmod 600 "$EXPORT_DIR.tar.gz.sha256"
   ```

9. Transfer the `.tar.gz` and `.sha256` files privately to the new VM's `~/scrap/`. Do not commit the bundle, dump, counts, credentials, or private payload to any repository.

## Destination restore boundary

Do not import automatically when the bundle arrives. First verify its checksum, inspect source/target PostgreSQL and Supabase versions, and back up the VM's current seeded fallback. Then choose the least-destructive restore sequence for the supplied dump set, restore Storage bytes, and compare source/target counts before accepting the migration.

The restore must remain strictly local. Do not use `--linked`, hosted database URLs, staging credentials, or production credentials.

## Acceptance Criteria

- [ ] The old source stack is preserved and no reset/prune command ran there.
- [ ] `roles.sql`, `schema.sql`, `data.sql`, `postgres-full.dump`, `storage.tar.gz`, `versions.txt`, and `counts.txt` are present in an owner-only bundle.
- [ ] The transfer archive checksum verifies on the new VM.
- [ ] A pre-import backup of the VM's seeded fallback exists.
- [ ] Database and Storage restore complete without touching a hosted project.
- [ ] Source and destination key-table counts match, including `auth.users`, `public.organizations`, and `storage.objects`.
- [ ] The owner-selected founder account's expected data and authenticated browser workflow are verified on the VM.
- [ ] The original machine and export bundle remain available until owner acceptance.

## Non-Goals

- Copying production or staging data.
- Reusing the x86 Docker volume directly on ARM64.
- Committing dumps, Storage objects, credentials, or row-level private data.

## Links

- `koder/docs/MIGRATION.md`
- `koder/STATE.md`
- `../code/site/supabase/config.toml`
- `../code/site/supabase/seed.sql`
