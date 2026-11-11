---
id: tech-python-uv-001
title: uv 入门：Python 环境、依赖管理与 pip 兼容命令
category: technical
tags: [Python, uv, pip, 虚拟环境, pyproject.toml, uv.lock, 离线安装, Windows]
status: maintained
created: "2026-11-11"
updated: "2026-11-11"
platform: "Windows / Linux / macOS"
source_project: ""
source_url: "https://docs.astral.sh/uv/"
sensitivity: public
---

# uv 入门：Python 环境、依赖管理与 pip 兼容命令

> 速记：**安装到环境用 `uv pip install`，记录项目依赖用 `uv add`，运行项目用 `uv run`。** uv 是 Astral 开发的 Python 项目与包管理工具，不是 Python 解释器，也不是 `pip` 的简单别名。

## 1. uv 是什么？

uv 是使用 Rust 实现的 Python 工具链，可管理 Python 版本、虚拟环境、依赖、项目及命令行工具。它将传统上由 pip、venv、部分版本管理和项目管理工具完成的工作整合起来。

| 工具 | 主要职责 |
| --- | --- |
| Python | 执行 Python 代码 |
| pip | 向 Python 环境安装包 |
| venv | 创建虚拟环境 |
| uv | 管理 Python、虚拟环境、包和项目依赖 |
| pytest | 执行 Python 测试 |

## 2. 安装与验证

Windows 可从官方文档选择 PowerShell 安装脚本、winget 等方式。建议先核对来源，再执行安装。已安装 uv 时运行：

```powershell
uv --version
uv python list
uv python install 3.11
```

`uv python install 3.11` 会在需要时下载受支持的 Python 发行版；在完全离线环境中不能假设可用。系统已经安装的 Python 也可以被 uv 发现并使用。

官方安装文档：https://docs.astral.sh/uv/getting-started/installation/

## 3. 最常用的命令

| 命令 | 作用 |
| --- | --- |
| `uv init my-project` | 创建新项目及项目配置 |
| `uv venv` | 在当前目录创建 `.venv` 虚拟环境 |
| `uv venv --python 3.11` | 使用 Python 3.11 创建虚拟环境 |
| `uv add requests` | 添加并记录项目依赖，更新锁文件 |
| `uv remove requests` | 移除项目依赖 |
| `uv sync` | 按项目配置及锁文件同步环境 |
| `uv lock` | 更新项目锁文件 |
| `uv run python main.py` | 在项目环境中运行脚本 |
| `uv run pytest` | 在项目环境中执行 pytest（需安装该依赖） |
| `uv pip install requests` | 用 pip 兼容接口直接安装包 |
| `uv pip list` | 列出目标环境的包 |
| `uv pip show requests` | 查看目标环境中某个包的信息 |
| `uvx ruff --version` | 临时运行工具（`uv tool run` 的别名） |

## 4. uv、pip 和 Python 的关系

**错误写法：** `uvpython -m pip install requests`。uv 没有 `uvpython` 这个命令。

**常用写法：**

```powershell
# 使用 uv 自带的 pip 兼容接口
uv pip install requests

# 使用指定解释器内安装的传统 pip（该解释器须安装 pip）
python -m pip install requests

# 运行由 uv 管理的项目 Python
uv run python --version
```

`uv pip` 并非在内部调用 `python -m pip`。uv 创建的虚拟环境通常**不默认安装传统 pip**，因此 `uv run python -m pip` 可能提示 `No module named pip`，这不代表 uv 故障。

## 5. 虚拟环境与项目管理

### 5.1 单纯使用虚拟环境

```powershell
uv venv --python 3.11
uv pip install requests
uv pip list
.\.venv\Scripts\python.exe --version
```

在 Windows PowerShell 下，`.venv\Scripts\python.exe` 是该环境的解释器。Linux/macOS 通常为 `.venv/bin/python`。不一定需要手动激活虚拟环境。

### 5.2 正式项目推荐流程

```powershell
uv init demo-uv
cd demo-uv
uv add requests
uv add --dev pytest
uv run python --version
uv run pytest
```

`pyproject.toml` 记录项目及依赖声明，`uv.lock` 记录解析后的依赖版本。应用项目通常应把锁文件纳入版本控制，`.venv` 不应提交。

### 5.3 `uv pip install` 与 `uv add` 的关键区别

- `uv pip install`：直接修改目标环境，不自动将依赖写入项目的 `pyproject.toml`。
- `uv add`：修改项目依赖声明并解析、锁定、同步。
- `uv run`：默认可能同步项目环境；先前仅用 `uv pip install` 手工装入、但未声明的包可能被移除。
- 要临时避免同步可用 `uv run --no-sync python main.py`，但长期仍建议规范声明项目依赖。

## 6. CST Studio Suite 2025 的离线安装案例

用户遇到的命令：

```powershell
pip install --no-index --find-links "path" cst-studio-suite-link
```

改为 uv 兼容接口：

```powershell
# 在 CST 自动化工程目录操作
uv venv --python 3.11

# 修改为实际存在、包含兼容安装包及所需依赖的目录
uv pip install --no-index --find-links "D:\offline-wheels" cst-studio-suite-link

# 检查分发包是否安装
uv pip show cst-studio-suite-link

# 确认目标 Python 环境
.\.venv\Scripts\python.exe --version
```

**重要边界：** `D:\offline-wheels` 是示例目录，不是已经核实的 CST 安装位置；Python 3.11 只是示例版本，应按 CST 接口包支持情况选择。安装成功不代表能连接 CST 或保存工程，仍需在真实 CST 2025 上进行接口调用与回读测试。

`--no-index` 表示不查询包索引，`--find-links` 表示在指定目录查找候选安装包。离线依赖必须预先备齐。详见 [pip 安装命令与离线安装速查](pip-安装与离线安装.md) 和 [CST Studio Suite 入门](../仿真与CAE/CST/CST-Studio-Suite.md)。

## 7. 常见问题

| 现象 | 可能原因与处理 |
| --- | --- |
| `uv` 不是内部或外部命令 | 未安装、PATH 未刷新，重新打开终端并检查 |
| `No virtual environment found` | 当前目录无 `.venv`，先执行 `uv venv` 或明确目标 Python |
| `No module named pip` | uv 环境不默认包含传统 pip，优先用 `uv pip` |
| 安装成功但运行时找不到包 | 安装环境与运行环境不同；检查解释器路径 |
| `uv run` 后手工安装的包消失 | 项目同步可能移除未声明依赖；使用 `uv add` |
| 离线安装找不到包 | 本地文件缺失、Python/平台不兼容或依赖不完整 |
| `uv python install` 无法下载 | 网络不可用；使用已有解释器或提前准备安装资源 |

## 8. 推荐实践

1. 日常新项目：`uv init` → `uv add` → `uv run`。
2. 临时实验或本地安装包：`uv venv` → `uv pip install`。
3. 多 Python 版本：明确指定 `--python`，不要依赖不确定的系统默认版本。
4. 离线企业环境：提前准备 Python 解释器、wheel 及依赖，确认架构和版本匹配。
5. 将 `pyproject.toml` 与 `uv.lock` 作为项目可复现的依赖依据；不要提交 `.venv`。

## 9. 参考资料

- [uv 官方文档](https://docs.astral.sh/uv/)
- [uv 的 pip 兼容接口](https://docs.astral.sh/uv/pip/)
- [uv 项目管理](https://docs.astral.sh/uv/guides/projects/)
- [pip 离线安装速查](pip-安装与离线安装.md)
