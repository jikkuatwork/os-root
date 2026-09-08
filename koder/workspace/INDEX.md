# OneSource Workspace Registry

## Contract

`root` is the organization control plane. The parent directory is the portable workspace boundary:

```text
Onesource/
├── root/          # harness entrypoint and cross-project state
├── code/site/     # primary product
├── grants/lens/   # LENS grant workspace
└── ...            # independent supporting, archived, and reference repos
```

- Open every coding harness from `root/`; do not start a separate harness in a child repository.
- Resolve all managed paths relative to `root/`. Never persist `/home/<user>` in control-plane artifacts.
- Keep each child as an independent Git repository. Its source, commits, issue state, and local instructions remain authoritative there.
- Use root artifacts only for organization-wide coordination or work spanning repositories. Link to child artifacts instead of copying them.
- Do not convert the workspace to submodules, move repositories, or consolidate histories without an explicit owner decision and migration plan.

## Registry

`repos.tsv` is the canonical inventory of sibling Git repositories. Its tiers mean:

- `focus` — read the handoff at root session open; currently the product and grant workspaces.
- `support` — first-party supporting material or tooling; load only when the task reaches it.
- `archive` — retained historical repository; do not modernize or clean incidentally.
- `reference` — external/reference clone; treat upstream content as read-only unless explicitly asked.

`../root` is intentionally omitted from `repos.tsv`; `workspace-status` always checks it separately.

Run:

```bash
./koder/bin/workspace-status          # root + focus repos, with whole-registry summary
./koder/bin/workspace-status --all    # every registered repo
```

The command is read-only. It exits non-zero only for inventory drift such as a missing path, a non-repository path, or an unregistered Git repository. Dirty repositories are reported but do not make the inventory invalid.

## Working in a child repository

Before edits:

1. Read the target's `AGENTS.md` or `CLAUDE.md` and `koder/STATE.md` when present.
2. Inspect its working tree and index. If intended paths are already dirty, stop and coordinate.
3. Identify whether the work is product-only, local-only, shared infrastructure, or production-affecting.

During and after edits:

- Run commands from the root harness with `git -C <path>` or a bounded `(cd <path> && ...)` subshell.
- Apply the target's validators, scratch gate, commit policy, and production restrictions.
- Commit in the target repository; never stage child paths into `root`.
- For multi-repository work, keep one explicit commit/result per repository and record those hashes in the root handoff at close.

## Adding or removing a repository

1. Make the repository change under `Onesource/`.
2. Add, update, or remove exactly one row in `repos.tsv`.
3. Run `./koder/bin/workspace-status --all`.
4. Update this index only when the management contract changes.
5. Commit the registry change in `root`.

## Non-repository workspace areas

The initial survey also found useful directories that are not independent Git roots, including `../media-kit`, `../prompts`, `../ivc-core-concept`, and `../code/primary-project-fork`. They are not protected by their own Git histories. Preserve them deliberately when migrating the whole workspace; do not silently treat them as registered repositories.
