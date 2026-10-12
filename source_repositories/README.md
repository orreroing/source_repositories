# 共享源码快照索引

这个 GitHub 仓库只保存源码快照的身份和 Commit 历史索引，不保存原始仓库的嵌套 `.git` 目录或 Git 对象。

## 目录约定

```text
manifest.jsonl
commit_benchmark_pipeline/inputs/commit_histories/
  <formal-snapshot-id>.commits.jsonl
  <formal-snapshot-id>.dependencies.json

extra/
  manifest.jsonl
  commit_benchmark_pipeline/inputs/commit_histories/
    <other-snapshot-id>.commits.jsonl
    <other-snapshot-id>.dependencies.json
```

顶层 `manifest.jsonl` 只包含正式选定的十个应用仓库。`extra/manifest.jsonl` 包含其余快照，包括正式仓库的旧快照、候补仓库、排除仓库和依赖快照。正式任务默认只解析顶层清单；任务若明确引用 `extra` 中的 `source_snapshot_id`，才读取额外清单。

Commit JSONL 保存 Commit、父提交、Tree 和修改文件等索引，不保存补丁。主仓库的依赖报告保存精确 gitlink Commit 和依赖快照 ID；依赖快照本身可以位于 `extra/`。只有完整 Git 对象归档同时存在时，才可以恢复源码并创建 worktree。

不要在这个索引仓库中执行上游仓库的 `git checkout <commit>`。重建程序应先读取对应清单，再从独立的 Git 对象归档下载或恢复源码，并用任务提供的 `base_commit` 验证对象存在。
