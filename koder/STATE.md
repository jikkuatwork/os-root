---
updated_at: "08 Sep 2026 | 09:09 PM IST"
---

# Koder State

## Past

- Initialized the koder-pattern scaffold in `e437b61` and the OneSource workspace control plane in `ae14cfe`.
- Registered all 27 sibling Git repositories while preserving their independent histories; designated `../code/site` and `../grants/lens` as focus repositories.
- Added root-only harness policy, cross-repository open/close handling, read-only inventory checks, and the M4 Air VM migration runbook.

## Present

- `root` is the only harness entrypoint; target commands run through explicit sibling paths and target-local rules remain authoritative.
- Registry health is `27/27` present with no missing or unregistered Git roots. Both focus repositories are clean.
- Seven non-focus repositories had pre-existing changes at adoption: `agreement`, `archive-blogs`, `archive-categorisation`, `archive-os-slides-old`, `archive-team`, `archive-vercel-old`, and `figma-frames`; preserve them unless explicitly assigned.
- This control repository has no remote or upstream yet.

## Future

- Create a private remote for `root` and push its history before relying on it as the sole control plane.
- Before migration, resolve or deliberately preserve dirty repositories, transfer secrets separately, decide whether local Supabase data needs export, and remove regenerable `node_modules/` plus `.next/` trees.
- On the ARM64 VM, restore the full `Onesource/` layout, run `koder/bin/workspace-status --all`, reinstall from lockfiles, recreate the local Docker/Supabase stack, and validate the site.
