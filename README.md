# HarmonyOS Source Snapshot Index

This repository is the GitHub metadata mirror for the HarmonyOS benchmark source snapshots. It stores `manifest.jsonl`, the commit-history JSONL files, and dependency-capture reports. It does not store the nested `.git` directories from the source repositories.

The complete Git objects, immutable `refs/bench-snapshots/*` refs, and working trees are backed up at:

```text
orrero@172.16.108.11:/data/home/orrero/wzh/window_store/harmony_git_history_backup_20260911
```

## Pulling A Snapshot

Clone this repository to inspect snapshot identities and commit indexes:

```powershell
git clone https://github.com/orreroing/source_repositories.git source_repositories-index
Get-Content source_repositories-index\manifest.jsonl | Select-Object -First 1
```

Use the full `source_snapshot_id` from `manifest.jsonl`. The GitHub mirror is an index only, so `git checkout <commit>` cannot restore an upstream commit from this repository. Restore the matching snapshot from the fixed-server backup, then resolve the repository through the manifest before creating a worktree.

## Updating The Mirror

When a new snapshot is captured locally, copy the updated `manifest.jsonl`, `commit_benchmark_pipeline/inputs/commit_histories/`, and README changes into this repository. Commit and push those metadata changes to GitHub. Sync the complete snapshot separately to the fixed-server backup and record the SHA-256 of `manifest.jsonl` in the commit message or release notes.

```powershell
$mirror = 'E:\path\to\source_repositories-index'
Copy-Item source_repositories\manifest.jsonl "$mirror\manifest.jsonl" -Force
New-Item -ItemType Directory -Force "$mirror\commit_benchmark_pipeline\inputs\commit_histories" | Out-Null
Copy-Item commit_benchmark_pipeline\inputs\commit_histories\* "$mirror\commit_benchmark_pipeline\inputs\commit_histories" -Force
git -C $mirror add README.md manifest.jsonl commit_benchmark_pipeline\inputs\commit_histories
git -C $mirror commit -m "Update source snapshot index"
git -C $mirror push origin main
```

The current mirror was prepared from the local snapshot registry. It contains 85 snapshot records and 108 commit/dependency index files.

The detailed repository contract and capture commands are also available at [`source_repositories/README.md`](source_repositories/README.md).
