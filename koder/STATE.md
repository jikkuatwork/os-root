---
updated_at: "09 Sep 2026 | 01:23 PM IST"
---

# Koder State

## Past

- Initialized koder-pattern in `e437b61`, the 27-repository control plane in `ae14cfe`, and the persistent-VM bootstrap in `bbacb47`.
- Restored the pinned `os.zip` payload on the ARM64 VM: all 27 sibling Git roots are present and `git fsck` passes for every repository.
- Hardened transfer privacy in `69bf76a`: bootstrap now rejects broadly readable archives and restricts restored payload permissions to the owner.
- Installed the site lockfile, then passed 81 Vitest files / 325 tests, typecheck, and a production build across 118 routes.
- Started a fresh local Supabase stack, applied 99 migrations and all six configured seed files, and verified the local REST API and seeded PostgreSQL data.
- Fixed VM-network login in site commits `15a49cb` and `32d2d90`; a real Chromium login now reaches the seeded fund dashboard without console or request errors.
- Confirmed the transfer did not include the old local database or Storage volume and filed the portable source-export handoff at `koder/issues/001_restore_original_local_supabase/INDEX.md`.

## Present

- `root` remains the only harness entrypoint. `root`, `../code/site`, and `../grants/lens` are clean; root and site handoffs are synchronized to their `origin` remotes.
- The site is live through user unit `onesource-site-dev.service` at `http://localhost:3000`; the verified VM-network endpoint is discoverable from the Next.js service log.
- Twelve local Supabase containers are healthy through rootless Podman. Studio is at `http://127.0.0.1:54323` and local mail is at `http://127.0.0.1:54324`.
- The site's private `.env.local` targets the VM-reachable app and Supabase endpoints, and its local API keys match the running stack. No hosted database, cloud resource, deployment, or production data was touched.
- Current database content is migration-and-seed output, not a 1:1 copy. Issue `001` is blocked on exporting the preserved old-machine local database and Storage objects.
- Seven preserved repositories retain their source-machine changes: `agreement`, `archive-blogs`, `archive-categorisation`, `archive-os-slides-old`, `archive-team`, `archive-vercel-old`, and `figma-frames`.
- With app and database running, the 6-CPU / 15-GiB VM retained about 11 GiB available RAM. The Projects volume has about 50 GiB free; the root filesystem has about 12 GiB free after 9.6 GB of container images.
- `~/scrap/os.zip` remains owner-only (`0600`). Supabase CLI `2.117.0` was run through pinned `pnpm dlx`; no global package was installed.

## Future

- On the old machine, follow `koder/issues/001_restore_original_local_supabase/INDEX.md` to export portable SQL, a full fallback dump, Storage objects, versions, counts, and checksums without resetting the source stack.
- Transfer that private bundle to this VM, then validate and restore it locally before deleting either source or archive.
- Keep an eye on root-disk growth because Podman stores images under the non-Projects home filesystem; relocate its graph root only through a deliberate migration if more images are needed.
- After a reboot, start the user Podman socket and local Supabase again, then launch the site from `../code/site`; the current services remain running for this VM session.
- Do not alter the seven preserved dirty repositories unless explicitly assigned.
