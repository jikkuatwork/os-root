---
updated_at: "08 Sep 2026 | 11:58 PM IST"
---

# Koder State

## Past

- Initialized koder-pattern in `e437b61`, the 27-repository control plane in `ae14cfe`, and the persistent-VM bootstrap in `bbacb47`.
- Restored the pinned `os.zip` payload on the ARM64 VM: all 27 sibling Git roots are present and `git fsck` passes for every repository.
- Hardened transfer privacy in `69bf76a`: bootstrap now rejects broadly readable archives and restricts restored payload permissions to the owner.
- Installed the site lockfile, then passed 81 Vitest files / 325 tests, typecheck, and a production build across 118 routes.
- Started a fresh local Supabase stack, applied 99 migrations and all six configured seed files, and verified the local REST API and seeded PostgreSQL data.

## Present

- `root` remains the only harness entrypoint. `root`, `../code/site`, and `../grants/lens` are clean; this handoff leaves root two commits ahead of `origin/master`.
- The site is live through user unit `onesource-site-dev.service` at `http://localhost:3000`; the verified VM-network endpoint is discoverable from the Next.js service log.
- Twelve local Supabase containers are healthy through rootless Podman. Studio is at `http://127.0.0.1:54323` and local mail is at `http://127.0.0.1:54324`.
- `.env.local` targets `127.0.0.1:54321` and its local API keys match the running stack. No hosted database, cloud resource, deployment, or production data was touched.
- Seven preserved repositories retain their source-machine changes: `agreement`, `archive-blogs`, `archive-categorisation`, `archive-os-slides-old`, `archive-team`, `archive-vercel-old`, and `figma-frames`.
- With app and database running, the 6-CPU / 15-GiB VM retained about 11 GiB available RAM. The Projects volume has about 50 GiB free; the root filesystem has about 12 GiB free after 9.6 GB of container images.
- `~/scrap/os.zip` remains owner-only (`0600`). Supabase CLI `2.117.0` was run through pinned `pnpm dlx`; no global package was installed.

## Future

- Inspect the application locally and exercise the seeded login/workflows before deleting the source machine or transfer ZIP.
- Keep an eye on root-disk growth because Podman stores images under the non-Projects home filesystem; relocate its graph root only through a deliberate migration if more images are needed.
- After a reboot, start the user Podman socket and local Supabase again, then launch the site from `../code/site`; the current services remain running for this VM session.
- Review and push the two root commits when ready; do not alter the seven preserved dirty repositories unless explicitly assigned.
