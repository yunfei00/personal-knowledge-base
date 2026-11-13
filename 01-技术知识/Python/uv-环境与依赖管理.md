---
id: tech-python-uv-001
title: uv 入门：Python 环境、依赖管理与 pip 兼容命令
category: technical
tags: [Python, uv, pip, 虚拟环境, pyproject.toml, uv.lock, 可选依赖, extra, dependency-groups, 离线安装, Windows]
status: maintained
created: "2026-11-11"
updated: "2026-11-13"
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
- `uv run`：运行前通常会确认项目环境的依赖可用，默认采用非精确同步，一般保留额外安装的包；`uv sync` 默认精确同步，可能移除没有声明或未选中的包。
- 要临时避免同步可用 `uv run --no-sync python main.py`，但长期仍建议规范声明项目依赖。

### 5.4 `--extra`：按需启用项目的可选依赖（2026-11-13 补充）

**一句话记住：`--extra viewer` 只是要求 uv 把 `viewer` 这组*已在项目中声明的可选依赖*准备好，并不代表启动查看器或执行某个叫 viewer 的程序。** 真正运行什么，由命令末尾的 `python 脚本.py` 决定。

#### ① `extra` 定义在哪里？

它在项目根目录的 `pyproject.toml` 内通过 `[project.optional-dependencies]` 定义。例如 CST 自动识别项目：

```toml
[project.optional-dependencies]
cad = ["cadquery-ocp>=7.9,<9"]

viewer = [
    "cadquery-ocp>=7.9,<9",
    "PySide6>=6.7,<7",
    "pyvista>=0.45,<1",
    "pyvistaqt>=0.11,<1",
]
```

- `cad`：只启用 Open CASCADE 精确几何计算相关依赖。
- `viewer`：启用 Open CASCADE 和 PySide6/VTK 三维查看器依赖。
- 名称 `viewer`、`cad` 是**项目自己定义的**，不是 uv 内置的固定选项。`pyproject.toml` 没有声明对应 extra，就不能把它当作现成的依赖组使用。
- `extra` 的作用是**选择安装哪些包**。是否运行 GUI，由被调用的 Python 程序决定。

#### ② `uv run --extra` 与 `uv sync --extra` 的区别

| 命令 | 含义 |
| --- | --- |
| `uv sync --extra viewer` | 同步项目环境，安装默认依赖及 `viewer` 可选依赖；不运行脚本 |
| `uv run --extra viewer python view_solids.py "D:\CST\spring.step"` | 确保本次执行具有 `viewer` 依赖，并启动三维查看器脚本 |
| `uv run --extra cad python run_project.py "D:\CST\old.cst" "D:\CST\new.cst"` | 使用 `cad` 几何依赖执行后台 CST 处理，不要求装 GUI 依赖 |
| `uv run python main.py` | 在项目环境运行，默认不主动选择额外的 optional extras |
| `uv sync --all-extras` | 同步所有可选依赖，适用于需要全部功能的开发环境 |
| `uv run --extra cad --extra viewer python main.py` | 同时选择多个已声明的 extra |

下面是一组 Windows PowerShell 示例（**在含有 `pyproject.toml` 的项目根目录运行**；CST 文件路径仅为示例）：

```powershell
# 查看当前项目可选依赖：打开 pyproject.toml，检查 [project.optional-dependencies]

# 只为后台几何处理准备环境，不启动程序
uv sync --extra cad

# 使用几何分析依赖执行主程序
uv run --extra cad python run_project.py "D:\CST\Original.cst" "D:\CST\Completed.cst"

# 准备含 PySide6/VTK 的三维显示环境
uv sync --extra viewer

# 真正启动三维查看器：最后的 view_solids.py 才决定打开窗口
uv run --extra viewer python view_solids.py "D:\CST\SpringContact.step"

# 安装项目中声明的全部 optional extras（一般开发调试才需要）
uv sync --all-extras
```

**注意**：`uv run` 在运行前会确保所需依赖可用；`uv sync` 默认是**精确同步**，可能移除本次没有选择的可选依赖以及环境里额外安装的包。`uv run` 默认采用**非精确同步**，一般不会清掉其他已安装的包，但仍要确保当前执行明确选择需要的 extra。已用 `--no-sync` 时，uv 跳过同步，缺少依赖不会被自动安装。

#### ③ `--extra`、`--group`、`--with` 不要混淆

| 选项 | 读取/使用的依赖 | 典型用途 |
| --- | --- | --- |
| `--extra viewer` | `[project.optional-dependencies]` 中的 `viewer` | 安装 GUI 等**项目可选功能**所需依赖 |
| `--group dev` | `[dependency-groups]` 中的 `dev` | 安装 pytest、ruff 等**开发依赖** |
| `--with rich` | 本次命令临时增加包 `rich` | 临时运行需要额外包的工具/脚本，不必修改项目声明 |

例如：

```powershell
# 给项目正式登记一组可选依赖（会更新 pyproject.toml）
uv add --optional viewer PySide6 pyvista pyvistaqt

# 给 dev 开发组添加测试工具（不是 extra）
uv add --dev pytest ruff

# 测试时显式启用开发组 + 三维查看器 extra
uv run --group dev --extra viewer pytest

# 临时给这次运行加入 rich，不将 rich 写进项目依赖
uv run --with rich python main.py
```

`uv add --optional viewer` 用来**声明/维护**可选依赖；`uv run --extra viewer` 用来**使用**已声明的依赖。两者不是一回事。`dev` 组通常会被 uv 默认同步，但可由项目配置更改，是否带 `--group dev` 可按项目实际配置决定。

#### ④ 常见误区与排查

- **把 `--extra viewer` 当成“打开窗口”**：错。它只选依赖；应运行 `view_solids.py` 这样的 GUI 脚本。
- **`uv run --extra viewer` 提示找不到 extra**：先确认当前工作目录属于正确项目，再检查 `pyproject.toml` 中有无 `[project.optional-dependencies]` 和 `viewer`。
- **已经 `uv sync --extra viewer`，却无法 `import PySide6`**：核查是否进入正确项目、解释器是否使用该项目 `.venv`、安装日志是否报错。
- **只需要后台几何计算**：优先用 `--extra cad`，不必为后台命令安装 PySide6/VTK。
- **离线企业环境**：`--extra` 不等于离线安装；所需 Python 包及依赖必须已经缓存或有本地安装源。
- **先 `uv pip install` 再 `uv sync`**：精确同步可能移除手工安装且未声明的包，项目长期依赖应登记在 `pyproject.toml`。

官方依据：[项目可选依赖](https://docs.astral.sh/uv/concepts/projects/dependencies/#optional-dependencies) · [锁定与环境同步](https://docs.astral.sh/uv/concepts/projects/sync/#syncing-optional-dependencies) · [uv run 命令](https://docs.astral.sh/uv/reference/cli/#uv-run)。

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
| `uv sync` 后手工安装的包消失 | `uv sync` 默认精确同步，可能移除未声明依赖；长期依赖应使用 `uv add` |
| 离线安装找不到包 | 本地文件缺失、Python/平台不兼容或依赖不完整 |
| `uv python install` 无法下载 | 网络不可用；使用已有解释器或提前准备安装资源 |
| `uv run --extra viewer` 报错 | 检查当前项目的 `[project.optional-dependencies]` 是否声明 `viewer` |

## 8. 推荐实践

1. 日常新项目：`uv init` → `uv add` → `uv run`。
2. 临时实验或本地安装包：`uv venv` → `uv pip install`。
3. 多 Python 版本：明确指定 `--python`，不要依赖不确定的系统默认版本。
4. 离线企业环境：提前准备 Python 解释器、wheel 及依赖，确认架构和版本匹配。
5. 将 `pyproject.toml` 与 `uv.lock` 作为项目可复现的依赖依据；不要提交 `.venv`。
6. 对 GUI、CAD、AI 等重量级功能使用 `project.optional-dependencies` 拆分依赖，执行时按需选择 `--extra`。

## 9. 参考资料

- [uv 官方文档](https://docs.astral.sh/uv/)
- [uv 的 pip 兼容接口](https://docs.astral.sh/uv/pip/)
- [uv 项目管理](https://docs.astral.sh/uv/guides/projects/)
- [uv 可选依赖 extras](https://docs.astral.sh/uv/concepts/projects/dependencies/#optional-dependencies)
- [uv 同步策略与 `--extra`](https://docs.astral.sh/uv/concepts/projects/sync/)
- [uv 运行命令参考](https://docs.astral.sh/uv/reference/cli/#uv-run)
- [pip 离线安装速查](pip-安装与离线安装.md)
