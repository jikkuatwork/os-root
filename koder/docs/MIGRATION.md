# Moving the OneSource Workspace to an M4 Air VM

## Recommendation

Yes—the site can run in a VM on an M4 Air, including its local Supabase Docker stack, provided the guest is a native ARM64 Linux VM with Docker support and enough memory/disk. Rebuild dependencies and containers in the destination; do not copy machine-specific build products or Docker volumes.

Prefer a `tar` archive over ZIP because this workspace uses relative symlinks and executable scripts. Preserve the complete `Onesource/` directory layout so `root/` remains a sibling of `code/`, `grants/`, and the other areas.

## What the initial audit found

On 2026-09-08:

- The source environment was Linux `x86_64`; an M4-native VM will normally be `arm64`/`aarch64`.
- The workspace held 27 sibling Git repositories plus this control repository.
- Regenerable `node_modules/` directories occupied about `4.0G`.
- Regenerable `.next/` directories occupied about `3.6G`.
- `../code/site/node_modules` was about `1.1G`; `../code/site/.next` was about `2.6G`.
- The site used Node `24.8.0`, pnpm `10.33.0`, and Supabase CLI `2.90.0`.
- Local Supabase ran 11 Docker containers. Docker's images, containers, and database volumes live outside `~/Onesource/` and will not be captured by an archive of this directory.
- Site secrets existed in ignored `.env.local`, `.env.production`, and `.env.staging` files. Only `.env.example` was tracked.

Treat these as migration-time observations, not permanent version pins.

## Before creating the archive

1. Check every repository from the control root:
   ```bash
   cd ~/Onesource/root
   ./koder/bin/workspace-status --all
   ```
   Commit/push intentional work where a remote exists. For repositories without remotes, the archive is the only copy of local Git history, so keep a second verified backup.

2. Decide how secrets move. Do not place an unencrypted archive containing `.env*` files in email, public object storage, or an untrusted sync folder. Prefer recreating them from a password manager or transferring them in a separate encrypted package.

3. Decide whether local Supabase data is disposable:
   - If migrations and seeds reproduce everything needed, no Docker-volume transfer is required. On the destination, run `supabase start` and then `supabase db reset` against the fresh local stack.
   - If non-reproducible local data matters, export it separately and keep the dump private. For example:
     ```bash
     mkdir -p ~/onesource-private-migration
     chmod 700 ~/onesource-private-migration
     (cd ~/Onesource/code/site && \
       supabase db dump --local --data-only \
       --file ~/onesource-private-migration/site-local-data.sql)
     chmod 600 ~/onesource-private-migration/site-local-data.sql
     ```
     Test the restore plan before deleting the source VM. Never commit this dump. Do not use `supabase stop --no-backup` unless local data is intentionally disposable.

4. Stop development processes and the local Supabase stack cleanly after any export:
   ```bash
   (cd ~/Onesource/code/site && supabase stop)
   ```

5. Remove architecture-specific, regenerable directories. Preview first:
   ```bash
   find ~/Onesource -type d \( -name node_modules -o -name .next \) -prune -print
   ```
   After reviewing the list:
   ```bash
   find ~/Onesource -type d \( -name node_modules -o -name .next \) \
     -prune -exec rm -rf -- {} +
   ```
   Removing these directories is recommended, especially for an `x86_64` to ARM64 move. Lockfiles remain and dependencies will be reinstalled.

## Archive and verify

Create the archive outside the directory being archived:

```bash
cd ~
tar -czf "Onesource-$(date +%F).tar.gz" Onesource
sha256sum "Onesource-$(date +%F).tar.gz" > "Onesource-$(date +%F).tar.gz.sha256"
tar -tzf "Onesource-$(date +%F).tar.gz" >/dev/null
```

`tar` preserves the root repository's relative skill symlinks and executable bits. Encrypt the archive if it includes ignored environment files, private grant material, invoices, or other confidential data. Keep the source machine untouched until the destination passes validation.

If ZIP is mandatory, use an implementation/options that store symlinks as symlinks (Info-ZIP uses `-y`) and verify them after extraction. Do not assume a GUI ZIP tool preserves links or Unix modes.

## Destination VM baseline

Use a native ARM64 Linux guest rather than an emulated `x86_64` guest. A practical starting allocation for Next.js plus the local Supabase stack is 4 vCPUs, 8 GiB RAM, and at least 40 GiB free disk; 12 GiB RAM is more comfortable if the host has enough memory. Actual needs depend on which tests and builds run concurrently.

Install and verify:

- Git and SSH/GitHub authentication.
- Node and pnpm matching the intended project toolchain.
- Docker Engine plus the Compose plugin, with the current user allowed to access the daemon.
- Supabase CLI; begin with the source version for a low-drift migration, then upgrade deliberately.
- Any Vercel, Supabase, or other provider credentials through their normal login/secret setup—not by copying host credential stores blindly.

Docker/Supabase images normally used by this project are expected to run on Apple Silicon through an ARM64-capable engine, but the real acceptance test is a clean `supabase start`. A pinned image without an ARM64 variant may need emulation or a version adjustment.

## Restore and validate

```bash
cd ~
tar -xzf Onesource-YYYY-MM-DD.tar.gz
cd ~/Onesource/root
./koder/bin/workspace-status --all
```

Then restore site secrets securely and rebuild from lockfiles:

```bash
cd ~/Onesource/code/site
pnpm install --frozen-lockfile
supabase start
supabase db reset       # local stack only; destructive to that fresh local DB
pnpm test
pnpm typecheck
pnpm build
```

Finally run the site locally, check the key authenticated flows, and compare repository status with the source machine. Do not decommission the source until:

- the registry reports no missing/unregistered repositories;
- all required Git histories and ignored working files are present;
- root skill symlinks resolve;
- Docker/Supabase is healthy;
- the site installs, tests, builds, and starts;
- required secrets and provider logins work without exposing them in Git.
