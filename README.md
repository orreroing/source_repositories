# HarmonyOS Source Snapshot Index

This repository is the metadata mirror for the HarmonyOS benchmark source snapshots. The top-level index contains the ten formally selected application repositories. Other captured snapshots are kept under `extra/` and are excluded from the default benchmark source set.

The mirror stores snapshot manifests, commit-history JSONL files, and dependency-capture reports. It does not store the nested `.git` directories or Git objects from the source repositories. Complete Git objects, immutable `refs/bench-snapshots/*` refs, and working trees are backed up separately.

## Top-level formal set

The ten applications are CloudMusic, Jingmo, LinysBrowser, FlowCourse, ChatCube, Tieba Lite, CoolMallArkTS, Ehviewer, OHOTP Token, and SiYuan HarmonyOS.

- `manifest.jsonl` contains exactly the ten selected application snapshots.
- `commit_benchmark_pipeline/inputs/commit_histories/` contains their commit histories and their dependency-capture reports.
- A formal repository may reference a dependency snapshot in `extra/`; for example, the OHOTP history references the exact nanopb snapshot there.

## Extra snapshots

- `extra/manifest.jsonl` contains every snapshot outside the formal ten-repository set.
- `extra/commit_benchmark_pipeline/inputs/commit_histories/` contains the corresponding commit histories and dependency reports.
- `extra/README.md` documents the extra archive.

Use the full `source_snapshot_id` from the relevant manifest. This GitHub repository is an index only, so cloning it does not make `git checkout <commit>` possible for an upstream repository. Restore the matching Git objects from the separate source archive before creating a worktree.

The existing complete snapshot backup is located at:

```text
orrero@172.16.108.11:/data/home/orrero/wzh/window_store/harmony_git_history_backup_20260911
```

## Updating the mirror

When a formal snapshot changes, update the top-level `manifest.jsonl` and its history files. When a non-formal snapshot changes, update the matching files under `extra/`. Keep the two manifests separate and record the SHA-256 of the changed manifest in the release or commit notes.

The detailed capture contract is available in [`source_repositories/README.md`](source_repositories/README.md).
