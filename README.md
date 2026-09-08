# OneSource Workspace Root

This repository is the control plane for the repositories under `~/Onesource/`.
Open Pi, Codex, Claude, or any other coding harness **only from this directory**.
The harness may work in sibling repositories, but its session root stays here.

## Focus repositories

- `../code/site` — the primary OneSource product: a management platform for funds and startups.
- `../grants/lens` — the Singapore LENS grant study and delivery workspace, currently planning-led.

The complete repository registry is `koder/workspace/repos.tsv`. Check it with:

```bash
./koder/bin/workspace-status
./koder/bin/workspace-status --all
```

## Operating model

1. Start the session here and use the root `open` skill.
2. Select a target from `koder/workspace/repos.tsv`.
3. Before touching it, read that repository's `AGENTS.md`/`CLAUDE.md` and `koder/STATE.md` when present.
4. Keep the harness rooted here; run target commands with an explicit path, for example:
   ```bash
   git -C ../code/site status --short
   (cd ../code/site && pnpm test)
   ```
5. Test and commit product changes in the target repository. Use this repository for organization-wide routing, inventory, decisions, and handoff—not as a copy of child source trees.
6. End the session with the root `close` skill, checking every repository touched during the session.

Each sibling remains an independent Git repository with its own history and rules. Git cannot version files outside this root, so this control plane tracks their locations and health rather than absorbing their contents or turning them into submodules.

## Durable memory

- `koder/STATE.md` — cross-project session handoff.
- `koder/workspace/INDEX.md` — repository-management contract.
- `koder/issues/` — cross-project issues only; target-specific issues stay in the target repository.
- `CHANGELOG.md` — meaningful control-plane milestones.

## Moving to another machine

See `koder/docs/MIGRATION.md`. The short version: preserve the whole `Onesource/` layout, remove regenerable `node_modules/` and `.next/` directories before transfer, move secrets securely, and recreate Docker/Supabase state on the destination.
