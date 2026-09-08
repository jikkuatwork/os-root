---
updated_at: "08 Sep 2026 | 10:38 PM IST"
---

# Koder State

## Past

- Initialized koder-pattern in `e437b61`, the 27-repository control plane in `ae14cfe`, and the persistent-VM bootstrap in `bbacb47`.
- Designated `../code/site` and `../grants/lens` as focus repositories while preserving every sibling's independent Git history and local rules.
- Built and fully test-extracted the pinned `os.zip` transfer payload: 11 top-level workspace entries, 27 sibling Git roots, no `root/`, `node_modules/`, or `.next/` content.

## Present

- `root` is the only harness entrypoint; `origin` is `git@github.com:jikkuatwork/os-root.git`.
- Registry health is `27/27` present with no missing or unregistered Git roots. Both focus repositories are clean.
- Seven non-focus repositories retain pre-existing changes: `agreement`, `archive-blogs`, `archive-categorisation`, `archive-os-slides-old`, `archive-team`, `archive-vercel-old`, and `figma-frames`; preserve them unless explicitly assigned.
- The source Supabase stack is stopped with its Docker volume preserved outside the ZIP. All workspace `node_modules/` and `.next/` trees were removed.
- The transfer ZIP contains private ignored files and is not encrypted; protect it as a sensitive artifact. Its expected digest lives in `koder/workspace/TRANSFER.sha256`.

## Future

- Copy `os.zip` to VM path `~/scrap/os.zip`; only `~/Projects/` is persistent there.
- Clone `origin` to `~/Projects/Onesource/root`, run `./koder/bin/bootstrap-vm --check ~/scrap/os.zip`, then run the same command without `--check` to restore siblings and install site dependencies.
- Start the harness from the restored root and invoke `/open`; initialize fresh local Supabase separately or use bootstrap's `--with-supabase` option on first extraction.
- Keep the source machine and transfer ZIP until registry, secrets, tests, build, and local Docker/Supabase pass on the VM.
