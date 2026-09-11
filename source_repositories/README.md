# 共享源码仓库与 Commit 快照

这里是整个 benchmark 流程唯一的源码仓库存储位置。Commit 挖掘、编译验证、
初筛后的测试生成和最终 Gold 验证都复用同一份本地 Git 仓库。

## 数据规范

```text
source_repositories/
  manifest.jsonl
  <repo_id>/
    .git/
    <working tree>

commit_benchmark_pipeline/inputs/commit_histories/
  <snapshot_id>.commits.jsonl
```

- `repo_id`：`<source>__<owner>__<repository>`，只表示上游仓库身份。
- `snapshot_id`：`<repo_id>__<refs_digest前12位>`，绑定一次不可变 refs 快照。
- `manifest.jsonl` 只追加 `repository-snapshot/v1` 记录，旧记录和快照 refs 不覆盖。
- Commit JSONL 只保存 Commit、父提交、代码文件和代码行数等索引，不保存 Patch。
- 下游必须携带 `repo_id`、`source_snapshot_id`、`base_commit`、`commit`。
- 下游只能创建 `E:\ha` 下的临时 worktree，禁止 fetch、pull 或原地切换共享仓库。

含子模块的主仓库还需要保存依赖仓库快照。主仓库 Commit 中的 gitlink SHA
是唯一允许的依赖版本依据；默认按相同规则保存两层声明依赖。
只有当前 HEAD、没有 Commit 对象的源码副本，或根据编译结果猜出的近似版本，都不算
固定历史。

完整规范见
`commit_benchmark_pipeline/documentation/COMMIT_DATA_SPEC.md`。

当前用于后续实验的仓库和固定快照见
[documentation/EXPERIMENT_REPOSITORY_SET.md](../documentation/EXPERIMENT_REPOSITORY_SET.md)。`manifest.jsonl` 仍是快照身份的权威数据；
实验清单只记录选择范围和工具链分类。

该清单中的十个 `source_snapshot_id` 已标记为当前正式 Commit 历史。后续 Stage 1、Stage 2、Stage 3
和评测只能解析这些正式快照及其精确依赖；manifest 中其他快照只用于历史审计、回溯或候补评估，不能作为
新的任务构造输入。

## 捕获或更新仓库

只有下面的入口允许访问远端：

```powershell
python -u E:\harmony_project_all\commit_benchmark_pipeline\capture_repository_snapshot.py `
  --remote-url <git-url-or-local-repo> `
  --repo-id <source>__<owner>__<repository>
```

远端 refs 未变化时复用原 `snapshot_id`；refs 变化时新增 Snapshot 和 Commit
索引，不修改旧版本。这里不得放构建产物、Agent 输出、任务 Patch 或逐 Commit
源码副本。

捕获命令默认扫描该快照全部可达 Commit 的 `.gitmodules` 和 gitlink，捕获两层依赖
仓库及精确 Commit，并写出
`commit_benchmark_pipeline/inputs/commit_histories/<snapshot-id>.dependencies.json`。
存在无法获取的对象、缺 URL 的 gitlink 或不可访问仓库时，依赖清单标记为
`incomplete`，命令返回非零。可用 `--dependency-depth` 显式调整深度；默认值 `2`
避免把第三方依赖自身的测试、模糊测试和语料仓库无限展开。`--skip-dependencies`
只用于诊断，不得用于生成新任务的输入快照。

如果主仓库 gitlink 引用的依赖 Commit 已不在远端分支或标签上，但远端仍允许按
完整对象 ID 获取，可以重复传入 `--required-commit <40位SHA>`。这些对象会作为
`refs/required/*` 纳入摘要和不可变快照。远端拒绝任一对象时，本次捕获整体失败，
不会改用其他版本。

## 审计固定依赖历史

下面的只读命令检查指定主快照的全部可达 Commit，并确认每个 gitlink 的精确
Commit 是否存在于已登记的依赖快照中。它不会访问远端，也不会修改共享仓库。

```powershell
python -u E:\harmony_project_all\commit_benchmark_pipeline\code\scripts\audit_fixed_dependency_history.py `
  --source-snapshot-id <main-snapshot-id> `
  --output E:\ha\cr\<run-name>\fixed_dependency_history_audit.json
```

结果中的 `restorable` 表示 URL 能对应到已登记依赖仓库，并且精确 Commit 对象
存在；`missing_exact_dependency_history` 表示对象尚未保存或远端已不可恢复；
`gitlinks_without_url` 表示主仓库没有提供可验证的依赖来源。后两类不能通过替换为
相近版本来放行。

## GitHub 元数据镜像

`https://github.com/orreroing/source_repositories` 保存这套数据的可提交索引，不保存各仓库的嵌套 `.git` 目录。镜像内容包括 `manifest.jsonl`、Commit JSONL 和依赖审计 JSON；完整 Git 对象、snapshot refs 和工作树保存在固定服务器备份中：

```text
orrero@172.16.108.11:/data/home/orrero/wzh/window_store/harmony_git_history_backup_20260911
```

从 GitHub 拉取索引后，先确认 `manifest.jsonl` 中的 `source_snapshot_id`，再从固定服务器取得对应快照。不要在 GitHub 镜像中执行 `git checkout <commit>`，因为该镜像没有上游仓库的 Git 对象。

更新流程如下：

```powershell
git clone https://github.com/orreroing/source_repositories.git source_repositories-index
Get-Content source_repositories-index\manifest.jsonl | Select-Object -First 1
```

如果本地捕获了新的快照，只提交 `manifest.jsonl`、`commit_benchmark_pipeline/inputs/commit_histories/` 和 README 的变更到这个 GitHub 仓库；完整快照仍按同一个目录规则同步到固定服务器，并在提交说明中记录 `manifest.jsonl` 的 SHA-256。

```powershell
$mirror = 'E:\path\to\source_repositories-index'
Copy-Item source_repositories\manifest.jsonl "$mirror\manifest.jsonl" -Force
New-Item -ItemType Directory -Force "$mirror\commit_benchmark_pipeline\inputs\commit_histories" | Out-Null
Copy-Item commit_benchmark_pipeline\inputs\commit_histories\* "$mirror\commit_benchmark_pipeline\inputs\commit_histories" -Force
git -C $mirror add README.md manifest.jsonl commit_benchmark_pipeline\inputs\commit_histories
git -C $mirror commit -m "Update source snapshot index"
git -C $mirror push origin main
```
