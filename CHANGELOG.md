# Changelog

This is a concise, newest-first record of meaningful project changes. Keep
umbrella milestones rather than mirroring every commit. Current readiness belongs
in `koder/STATE.md`, live Git state, and current validation.

## 2026-09-09 — VM workspace restoration

- Restored all 27 sibling repositories, secured the private transfer payload, installed the site dependencies, and validated the ARM64 application build and local runtime.
- Initialized the local Supabase schema and seed baseline, restored LAN browser login, and confirmed the source Docker database and Storage volume were not part of `os.zip`.
- Filed `koder/issues/001_restore_original_local_supabase/INDEX.md` as the portable, private export handoff for the old machine and the guarded import boundary for this VM.

## 2026-09-08 — Persistent-VM bootstrap

- Added a fail-closed restore script for the split clone-plus-ZIP migration into `~/Projects/Onesource/`, including committed archive checksum verification, site dependency installation, and optional fresh local Supabase initialization.
- Hardened private-payload handling by rejecting broadly readable transfer archives and restricting restored files and directories to owner-only access before dependency installation.
- Taught root session opening to detect a fresh clone and surface the explicit bootstrap command without weakening `/open`'s read-only contract.

## 2026-09-08 — OneSource workspace control plane

- Registered every sibling Git repository under the single `root` harness entrypoint, with the product and LENS grant workspaces as bounded focus repositories.
- Added read-only repository health/inventory checks and cross-repository open/close rules while preserving every child as an independent Git root.
- Documented the x86-to-ARM migration path, cache cleanup, secret handling, and Docker/Supabase rebuild boundary.

## 2026-09-08 — Koder-pattern adoption

- Added durable operator handoff and shared session skills under `koder/`.
