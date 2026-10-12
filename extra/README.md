# Extra source snapshot archive

This directory contains captured repositories that are outside the ten formally selected HarmonyOS benchmark repositories. It includes old snapshots of selected repositories, candidate or excluded applications, system applications, sample collections, and their captured dependencies.

- `manifest.jsonl` is the index for this archive.
- `commit_benchmark_pipeline/inputs/commit_histories/` contains the matching Commit JSONL files and dependency reports.

These snapshots are retained for audit and historical lookup. They are not part of the default formal benchmark source set. A task may use an extra snapshot only when its task metadata explicitly names the corresponding `source_snapshot_id`.
