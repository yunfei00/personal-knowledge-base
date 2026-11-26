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

## 6. 保存临时修改

```bash
git stash push -u -m "before sync"
git stash list
git pull --ff-only
git stash apply
```

-u 包含未跟踪文件，但不包含忽略文件。确认恢复成功后使用 git stash drop 删除对应记录。stash pop 会在成功应用后删除记录；发生冲突时应检查状态。重要文件应另行备份。

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

## 8. 合并冲突

运行 git status 查看冲突文件，手动处理冲突标记，再 git add 对应文件。普通 merge 使用 git commit 完成，rebase 使用 git rebase --continue 完成当前步骤。放弃操作分别使用 git merge --abort 或 git rebase --abort。

## 9. 标签与版本

```bash
git tag
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
git show v1.0.0
```

标签不等于 GitHub Release。已发布标签不要随意移动。

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
