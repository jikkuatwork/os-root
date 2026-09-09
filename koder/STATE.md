---
updated_at: "09 Sep 2026 | 08:57 PM IST"
---

# Koder State

## Past

- Initialized koder-pattern in `e437b61`, the 27-repository control plane in `ae14cfe`, and the persistent-VM bootstrap in `bbacb47`.
- Restored and validated all 27 sibling Git roots on the ARM64 VM, hardened transfer privacy in `69bf76a`, and passed the site's 325 tests, typecheck, and 118-route production build.
- Initialized the VM's local Supabase schema and six seed files and verified a local authenticated browser workflow; that rebuilt database was not a copy of the source volume.
- On the source machine, exported the local `../code/site` PostgreSQL database to owner-only `~/Desktop/os.sql` as a complete plain schema-and-data dump.
- Verified the dump's PostgreSQL completion marker, 95 table definitions, 95 data blocks, `1,539,676`-byte size, and SHA-256 `daed4bb4b36f50289353ef9a0b12552e57ae4d491a6e06a6f96f7881f44c0cbb`.

## Present

- State: IN_PROGRESS — source database export complete; private transfer and destination restore pending.
- Source counts at export were 99 migrations, 25 auth users, 69 organizations, and 21 Storage metadata objects.
- The owner will place the transferred dump at `~/scratch/os.sql` on the new machine before the next session. This session has not observed or verified that destination file.
- `os.sql` contains private database schema and rows, including Auth and Storage metadata. It must remain outside Git and must not be printed into logs or chat.
- Physical Supabase Storage object bytes are not part of `os.sql`; database recovery must not be described as complete Storage recovery.
- Root and `../code/site` remain clean. The source local Supabase stack is running after the dump.
- Seven preserved repositories retain their source-machine changes: `agreement`, `archive-blogs`, `archive-categorisation`, `archive-os-slides-old`, `archive-team`, `archive-vercel-old`, and `figma-frames`.

## Future

- On the new machine, first verify `~/scratch/os.sql` is owner-only, exactly `1,539,676` bytes, and matches the recorded SHA-256; do not expose its contents.
- Confirm the destination is strictly local and create a pre-import backup of its current seeded database.
- Review a restore sequence for the full schema-and-data SQL before applying it; do not import blindly over populated seed tables and never use linked, staging, or production endpoints.
- After restore, compare the four source counts and verify the owner-selected account and expected data through the local browser workflow.
- Keep the source dump until owner acceptance, and handle physical Storage bytes separately only if the owner requires them.
- Use `koder/issues/001_restore_original_local_supabase/INDEX.md` as the authoritative restore handoff.
