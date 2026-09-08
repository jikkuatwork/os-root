# Restore OneSource on the Persistent M4 Air VM

## Transfer contract

The migration uses two independent pieces:

1. `git@github.com:jikkuatwork/os-root.git` supplies the control repository at `~/Projects/Onesource/root`.
2. `~/scrap/os.zip` supplies every sibling directory from the old `~/Onesource/` workspace.

The ZIP intentionally excludes `root/` so it cannot overwrite the fresh clone. It includes sibling `.git` directories, ignored working files, and non-repository areas, but excludes regenerable `node_modules/` and `.next/` trees. The exact transfer artifact is pinned by `koder/workspace/TRANSFER.sha256`; bootstrap verifies it before extraction, so copying only `os.zip` is sufficient.

Only `~/Projects/` is persistent on the destination VM. Do not restore the workspace to `~/Onesource/`; the bootstrap script derives `~/Projects/Onesource/` from the location of the cloned root.

## Important boundaries

- `/open` is observational. It reports missing repositories and the bootstrap command but never extracts files or installs packages itself.
- `koder/bin/bootstrap-vm` performs the one-time extraction and installs `../code/site` dependencies from `pnpm-lock.yaml`. It rejects an archive accessible to group or other users, then strips group/other permissions from the restored payload before installing dependencies.
- The archive contains private company/grant material and may contain ignored `.env.local`, `.env.production`, and `.env.staging` files. It is a sensitive transfer artifact: keep mode `0600`, transfer it directly, and delete or securely archive it after validation.
- Docker images, containers, and volumes live outside the source workspace and are not in the ZIP. A fresh local Supabase stack must be created on the VM.
- The source machine is `x86_64` and an M4-native guest is ARM64. Reinstalling dependencies is mandatory; copied native Node artifacts would not be trustworthy.

## Destination prerequisites

Use a native ARM64 Linux VM. Install:

- Git and GitHub SSH access;
- `unzip`;
- Node and pnpm (the source used Node `24.8.0` and pnpm `10.33.0` at transfer time);
- Docker Engine and Supabase CLI if local Supabase should be initialized immediately.

A practical starting allocation for Next.js plus Supabase is 4 vCPUs, 8–12 GiB RAM, and at least 40 GiB free disk.

## One-time restore

Copy `os.zip` to `~/scrap/os.zip`, then:

```bash
mkdir -p ~/Projects/Onesource
git clone git@github.com:jikkuatwork/os-root.git ~/Projects/Onesource/root
cd ~/Projects/Onesource/root

# Validate without changing the destination.
./koder/bin/bootstrap-vm --check ~/scrap/os.zip

# Extract every sibling repo/area and install site dependencies.
./koder/bin/bootstrap-vm ~/scrap/os.zip
```

To initialize a fresh local Supabase stack in the same one-time operation, use this instead of the final command:

```bash
./koder/bin/bootstrap-vm --with-supabase ~/scrap/os.zip
```

That option runs `supabase start` and then `supabase db reset` against the new VM's **local** database. If Docker/Supabase is not ready during extraction, omit the option and later run:

```bash
cd ~/Projects/Onesource/code/site
supabase start
supabase db reset
```

Do not run `bootstrap-vm` over an existing or partially restored workspace. It fails closed on any payload-path collision rather than merging unknown files.

## Open the restored workspace

After bootstrap succeeds:

```bash
cd ~/Projects/Onesource/root
# Start Pi/Codex/Claude here.
# Then invoke /open inside the harness.
```

`/open` checks all 27 registered sibling repositories, reads the focus handoffs for `code/site` and `grants/lens`, and reports the next task. No harness should be opened from a sibling repository.

## Acceptance checks

The bootstrap script verifies owner-only archive permissions, the committed SHA-256, ZIP CRCs, safe path layout, destination collisions, and the full repository registry. It also normalizes restored payload permissions to owner-only access. Before deleting the source machine, also verify:

```bash
cd ~/Projects/Onesource/root
./koder/bin/workspace-status --all

git -C ../code/site status --short
git -C ../grants/lens status --short

cd ../code/site
pnpm test
pnpm typecheck
pnpm build
```

Confirm that:

- all 27 sibling Git roots are present;
- the known pre-existing dirty repositories match the source handoff rather than new transfer damage;
- site dependencies install on ARM64;
- ignored environment files needed for local work are present and remain untracked;
- Docker/Supabase starts successfully if required;
- root skill symlinks resolve and `/open` runs from `~/Projects/Onesource/root`.
