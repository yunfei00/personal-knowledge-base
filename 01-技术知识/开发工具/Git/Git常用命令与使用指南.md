---
id: "TECH-GIT-20261126-01"
title: "Git 常用命令与使用指南"
category: technical
tags: [Git, GitHub, 版本控制, 开发工具]
status: draft
created: "2026-11-26"
updated: "2026-11-26"
platform: "Windows/Linux/macOS"
source_project: ""
source_url: "https://git-scm.com/docs"
sensitivity: public
---

# Git 常用命令与使用指南

## 1. 基本原理

Git 是分布式版本控制工具，GitHub 是远程代码托管平台。工作区通过 add 进入暂存区，通过 commit 进入本地仓库，通过 push 上传远程。fetch 只获取远程引用，pull 则获取并整合远程变更。

### 1.1 四个位置与远程引用

- 工作区：磁盘上的实际文件。
- 暂存区（index）：下一次提交的内容快照。
- HEAD：当前检出的提交，通常指向当前本地分支。
- origin/main：本地保存的远程跟踪引用，只有 fetch 等操作更新后才反映远端状态。
- upstream：本地分支配置的上游分支，决定无参数 pull 的默认来源。

origin 是远程仓库的惯用名称，不是 GitHub 的固定名称。main、dev 是示例分支名。以下命令应在目标仓库目录运行。尖括号表示需要替换的参数。

## 2. 初始化与配置

```bash
git --version
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --list
git clone <repository-url>
git init
git remote -v
git remote add origin <repository-url>
git remote set-url origin <repository-url>
```

以上是独立命令示例，尖括号内容需替换，不要整段盲目执行。已有远程仓库通常直接 clone，无需 init。

### 2.1 逐条解释

| 命令 | 参数与效果 |
| --- | --- |
| `git --version` | 查看 Git 版本，确认已安装；switch 需要 Git 2.23 或更新版本。 |
| `git config --global user.name "Your Name"` | 设置当前系统用户的默认提交作者名，不是 GitHub 登录名。 |
| `git config --global user.email "you@example.com"` | 设置默认提交邮箱；省略 --global 并在仓库内执行可设置仓库专用值。 |
| `git config --list` | 查看有效配置；加 --show-origin 可查看配置来源。 |
| `git clone URL` | 下载仓库、建立 origin、检出默认分支；通常无需再 init。 |
| `git init` | 在当前目录初始化本地仓库，不会自动创建 GitHub 仓库。 |
| `git remote -v` | 查看远程名称以及 fetch/push URL。 |
| `git remote add origin URL` | 新增名为 origin 的远程；若已存在则报错。 |
| `git remote set-url origin URL` | 修改已有 origin 的地址，不会自动下载代码。 |

## 3. 日常开发

```bash
git status -sb
git diff
git diff --staged
git add <file>
git add -A
git commit -m "feat: add feature"
git pull --ff-only
git push
git log --oneline --graph --all -20
git show HEAD
```

提交前检查暂存区，避免误提交密钥、环境配置和大文件。pull --ff-only 在历史分叉时拒绝整合，避免意外合并。

### 3.1 逐条解释

| 命令 | 参数与效果 |
| --- | --- |
| `git status -sb` | -s 简洁显示状态，-b 显示分支和上游；?? 为未跟踪文件。 |
| `git diff` | 查看已跟踪文件尚未暂存的变化，不显示未跟踪文件内容。 |
| `git diff --staged` | 比较暂存区与 HEAD，检查即将提交的变化。 |
| `git add 文件路径` | 将指定文件当前内容加入暂存区。 |
| `git add -A` | 暂存整个仓库新增、修改和删除的文件，默认排除未跟踪的忽略文件。 |
| `git commit -m "说明"` | 将暂存区生成本地提交；-m 指定提交说明，不会自动推送。 |
| `git pull --ff-only` | 获取并仅允许快进整合上游；历史分叉时失败，不自动产生合并提交。 |
| `git push` | 按配置推送本地提交，通常需要已设置上游；未提交文件不会上传。 |
| `git log --oneline --graph --all -20` | 单行显示提交、绘制分支关系、查看所有引用，最多显示 20 条。 |
| `git show HEAD` | 查看当前提交的说明及变更。 |

### 3.2 fetch、pull、push 的区别

`git fetch origin` 从 origin 获取对象及更新远程跟踪引用，不切换当前分支，也不合并工作区。它适合先了解远程变化再决定操作。可使用 `git log HEAD..origin/main --oneline` 查看远程 main 相对当前 HEAD 的新增提交。

`git pull` 相当于 fetch 后再按配置 merge 或 rebase；`git push` 则把本地提交发送到远程。三者不是同一个操作。

## 4. 分支管理

```bash
git branch -a
git switch main
git switch -c feature/demo
git push -u origin feature/demo
git fetch --prune origin
git switch main
git pull --ff-only
git merge feature/demo
git branch -d feature/demo
git push origin --delete feature/demo
```

删除远程分支前确认已合并且无人使用。团队项目优先通过 PR 和 CI 合并。

### 4.1 查看与切换分支

| 命令 | 参数与效果 |
| --- | --- |
| `git branch` | 查看本地分支，星号标记当前分支。 |
| `git branch -r` | 查看本地记录的远程跟踪分支，并非实时查询服务器。 |
| `git branch -a` | 同时查看本地及远程跟踪分支。 |
| `git branch -vv` | 显示本地分支提交、上游和领先/落后信息。 |
| `git branch --show-current` | 输出当前分支名称；detached HEAD 时为空。 |
| `git switch main` | 切换到 main，更新工作区；可能因本地修改冲突而拒绝。 |
| `git switch -` | 切换到上次检出的分支或位置。 |
| `git switch -c feature/demo` | 从当前 HEAD 创建新分支并立即切换；-c 表示创建。 |
| `git push -u origin feature/demo` | 推送指定分支并设置上游；-u 是 --set-upstream 的简写。 |
| `git fetch --prune origin` | 获取远程更新并清理已不存在的远程跟踪引用，不删除本地分支。 |
| `git fetch --all --prune` | 获取所有配置为参与 fetch --all 的远程并清理失效引用。 |
| `git merge feature/demo` | 将指定分支合入当前分支；执行前确认当前分支正确。 |
| `git branch -d feature/demo` | 安全删除已合并的本地分支；-D 强制删除，可能丢失未合并提交。 |
| `git push origin --delete feature/demo` | 删除服务器上的分支，不等于删除本地分支。 |

### 4.2 本地没有、远程已有的分支（常用）

假设 GitHub 已有 dev 分支，本地只有 main。推荐执行：

```bash
git fetch origin
git switch dev
git branch -vv
```

第一条更新本地 origin/dev 引用，但仍停留在 main。第二条在本地 dev 不存在、默认自动猜测开启且只有一个匹配远程（或配置了默认远程）时，自动创建 dev、关联 origin/dev 并切换工作区。第三条确认当前分支及上游。此时后续可以直接 git pull 和 git push。

更明确的写法（本地 dev 尚不存在）：

```bash
git fetch origin
git switch -c dev --track origin/dev
```

-c 创建本地 dev；--track 设置 origin/dev 为上游，同时以其提交为起点。多个远程存在同名分支时优先使用此写法。旧版 Git 可使用 git checkout -b dev --track origin/dev。

如果本地 dev 已存在，直接 git switch dev，再 git pull --ff-only；switch 本身不会更新已有本地分支到远程最新提交。若未关联上游，可执行 git branch --set-upstream-to=origin/dev dev。

如果 fetch 后仍看不到 origin/dev：

```bash
git remote -v
git ls-remote --heads origin dev
git config --get-all remote.origin.fetch
git fetch origin refs/heads/dev:refs/remotes/origin/dev
git switch -c dev --track origin/dev
```

前三条用于诊断远程地址、服务器是否有 dev、是否配置了单分支 fetch；确认服务器存在 dev 后，第四条显式获取该分支并建立远程跟踪引用。最后一条仅在本地 dev 不存在时执行。若本地修改妨碍切换，先 commit 或 stash，不要直接强制丢弃。

## 5. 强制同步远程代码

### 5.1 本地修改或未跟踪文件阻碍 pull

典型报错：The following untracked working tree files would be overwritten by merge。

如果明确不要本地修改和本地独有提交，且当前位于 main：

```bash
git status -sb
git fetch origin
git reset --hard origin/main
```

reset --hard 会覆盖已跟踪文件，并删除妨碍写入目标版本的未跟踪文件或目录；不会普遍删除其他未跟踪文件。不要误以为 git pull --force 可以强制覆盖工作区。

### 5.2 清理剩余未跟踪文件

先预览：

```bash
git clean -nd
```

确认列表后再执行：

```bash
git clean -fd
```

-f 强制执行，-d 包含目录。默认保留被忽略的文件。仅需删除特定冲突文件时，可使用 git clean -n -- <path> 预览，再用 git clean -f -- <path> 删除；目录需加 -d。

### 5.3 包含忽略文件的彻底清理（高风险）

```bash
git clean -ndx
```

确认后才执行 git clean -fdx。它可能删除 .env、虚拟环境、缓存和本地数据，不建议作为日常同步命令。

### 5.4 自动识别当前分支

PowerShell：

```powershell
$branch = git branch --show-current
if (-not $branch) { throw "当前不在分支上" }
git fetch origin
if ($LASTEXITCODE -ne 0) { throw "fetch 失败" }
git reset --hard "origin/$branch"
```

Bash：

```bash
branch=$(git branch --show-current)
[ -n "$branch" ] && git fetch origin && git reset --hard "origin/$branch"
```

假设远程名为 origin，且远程存在同名分支。执行前确认本地数据可丢弃。

### 5.5 强制同步逐条说明与验证

- git status -sb：确认当前分支和待处理文件。
- git fetch origin：刷新远程引用；失败时立即停止，不要使用旧引用继续 reset。
- git reset --hard origin/main：将当前本地分支指向 origin/main，同时重置暂存区及工作区。必须先确认当前分支是 main，否则会重置错误分支。
- git clean -nd：-n 仅预览，-d 包含未跟踪目录；不删除任何文件。
- git clean -fd：-f 执行删除，-d 包含目录；删除所有符合条件的未跟踪文件，而非仅冲突文件。
- git clean -ndx / -fdx：-x 额外包含忽略文件；前者预览，后者实际删除。
- PowerShell 的 $LASTEXITCODE 检查外部命令退出码；Bash 的 && 仅在前一步成功后执行下一步。

同步后执行：

```bash
git status -sb
git log -1 --oneline
git rev-parse HEAD
git rev-parse origin/main
```

最后两条哈希一致表示当前提交与本地记录的远程 main 一致；status 用于确认是否还有工作区变化。忽略文件可能仍存在，这是正常的。

## 6. 保存临时修改

```bash
git stash push -u -m "before sync"
git stash list
git pull --ff-only
git stash apply
```

-u 包含未跟踪文件，但不包含忽略文件。确认恢复成功后使用 git stash drop 删除对应记录。stash pop 会在成功应用后删除记录；发生冲突时应检查状态。重要文件应另行备份。

### 6.1 逐条解释

| 命令 | 参数与效果 |
| --- | --- |
| `git stash push -u -m "before sync"` | 保存已跟踪修改及未跟踪文件，-m 添加备注。 |
| `git stash list` | 查看保存记录，最新通常为 stash@{0}。 |
| `git stash show -p "stash@{0}"` | 查看指定记录的补丁；引号兼容 PowerShell。 |
| `git stash apply "stash@{0}"` | 恢复指定记录但保留记录；默认不保证恢复原暂存状态，可按需加 --index。 |
| `git stash pop` | 应用最新记录，成功后删除该记录。 |
| `git stash drop "stash@{0}"` | 删除指定记录，先确认不再需要。 |

stash 不是远程备份，也不保存本地独有提交；重要数据另行备份。

## 7. 撤销与恢复

| 命令 | 用途 |
| --- | --- |
| git restore <file> | 丢弃文件未暂存修改 |
| git restore --staged <file> | 取消暂存，保留工作区修改 |
| git reset --soft HEAD~1 | 撤销最近一次本地提交，保留暂存 |
| git reset --mixed HEAD~1 | 撤销最近一次本地提交，保留工作区 |
| git revert <commit> | 新建反向提交，适合已共享历史 |
| git reflog | 查找曾经的 HEAD 位置 |
| git branch rescue <commit> | 从找到的提交创建恢复分支 |

reflog 可能恢复曾提交的内容，但不能保证恢复 clean 删除的未跟踪文件或 reset 丢弃的未提交修改。

### 7.1 reset 三种模式

| 模式 | HEAD/当前分支 | 暂存区 | 工作区 |
| --- | --- | --- | --- |
| --soft | 移动到目标提交 | 保留 | 保留 |
| --mixed（默认） | 移动到目标提交 | 重置 | 保留 |
| --hard | 移动到目标提交 | 重置 | 重置已跟踪文件，可能删除阻碍写入的未跟踪文件 |

HEAD~1 表示当前提交沿第一父提交向前一代。仅在确实要撤销本地历史时使用 reset；已共享提交通常使用 revert。restore --staged 默认从 HEAD 恢复暂存区；普通 restore 默认从暂存区恢复工作区。

## 8. 合并冲突

运行 git status 查看冲突文件，手动处理冲突标记，再 git add 对应文件。普通 merge 使用 git commit 完成，rebase 使用 git rebase --continue 完成当前步骤。放弃操作分别使用 git merge --abort 或 git rebase --abort。

### 8.1 冲突处理示例

```bash
git status
git diff
git add path/to/resolved-file
git status
git merge --continue
```

手动编辑冲突文件并移除冲突标记后才执行 add；最后一条仅用于正在进行的 merge。若是 rebase，改用 git rebase --continue，可能需要重复处理多个提交。git merge --abort 和 git rebase --abort 分别取消对应操作，不要混用。

## 9. 标签与版本

```bash
git tag
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
git show v1.0.0
```

标签不等于 GitHub Release。已发布标签不要随意移动。

### 9.1 逐条解释

git tag 列出本地标签；git tag -a v1.0.0 -m "Release v1.0.0" 在当前提交创建附注标签（-a），并设置说明（-m）。git push origin v1.0.0 将指定标签上传；git show v1.0.0 查看标签及目标对象。git tag -d v1.0.0 删除本地标签；git push origin --delete tag v1.0.0 删除远程标签，发布后不应随意执行。

## 10. 常见问题速查

| 现象 | 建议 |
| --- | --- |
| non-fast-forward | fetch 后检查历史分叉，按团队约定 merge 或 rebase |
| untracked files would be overwritten | 备份冲突文件，或明确放弃本地数据后 reset --hard |
| local changes would be overwritten | commit、stash 或明确丢弃修改 |
| no upstream branch | git push -u origin <branch> |
| detached HEAD | git switch <branch>，若有新提交先创建恢复分支 |
| 已跟踪文件加入 gitignore 仍生效 | git rm --cached <file> 后提交，保留本地文件 |

不要将 force push 作为解决 pull 失败的手段。确需重写个人远程分支时，经协作确认后优先考虑 --force-with-lease；共享主分支禁止强推。

## 11. 参考资料

- [Git 官方命令手册](https://git-scm.com/docs)
- [Pro Git 中文版](https://git-scm.com/book/zh/v2)

本文为命令参考，未在实际仓库逐项执行验证。
