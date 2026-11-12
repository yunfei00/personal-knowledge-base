---
id: fix-pycharm-src-import-navigation-001
title: PyCharm 克隆 src 布局项目后导入无法跳转：设置 Sources Root 并重新加载解释器
category: troubleshooting
tags: [PyCharm, Python, src-layout, Sources Root, uv, editable install, 导入路径, 代码跳转]
status: verified
created: "2026-11-12"
updated: "2026-11-12"
platform: "Windows / PyCharm / Python 3.11 / uv"
source_project: "cst-material-automation"
source_url: "https://github.com/yunfei00/cst-material-automation"
sensitivity: public
---

# PyCharm 克隆 src 布局项目后导入无法跳转：设置 Sources Root 并重新加载解释器

> **结果：已由实际操作验证解决。** 关键操作是将 `src` 标记为 **Sources Root（源码根目录）**，然后在 PyCharm 中**重新加载项目 Python 解释器**。只做其中一步时，本次问题没有立即消失。

## 1. 现象

从 GitHub 克隆 Python 工程到本地并使用 PyCharm 打开后：

- 本地包导入语句出现波浪线。
- 按 Ctrl + 点击函数、类或导入符号，不能跳转到定义。
- 已执行 `uv pip install -e .`，但 PyCharm 中的跳转问题依旧存在。

发生于 [cst-material-automation](https://github.com/yunfei00/cst-material-automation) 工程，使用规范的 `src layout`：

```text
cst-material-automation/
├── pyproject.toml
├── src/
│   └── cst_material/
│       ├── __init__.py
│       ├── domain.py
│       └── geometry.py
└── tests/
    └── test_geometry.py
```

典型导入：

```python
from cst_material.domain import BoundingBox
from cst_material.geometry import box_volume, center
```

## 2. 本次已验证的解决步骤

1. 在 PyCharm 左侧 Project 目录树中，右键 **`src` 文件夹**。
2. 选择 **Mark Directory as → Sources Root**（标记目录为源码根目录）。
3. 进入 **Settings → Project → Python Interpreter**（不同 PyCharm 版本的菜单名称可能略有不同）。
4. 确认选择当前工程的解释器（通常为项目下 `.venv/Scripts/python.exe`），并让 PyCharm **重新加载 / 刷新该解释器**，等待索引完成。
5. 再次在 `tests/test_geometry.py` 中 Ctrl + 点击 `BoundingBox`、`box_volume`、`center`。

**实际结果：** 将 `src` 标记为源码根目录之后，问题仍存在；随后重新加载解释器，代码跳转恢复正常。

## 3. 为什么这样处理？

该工程的包位于 `src/cst_material`，而不是仓库根目录的 `cst_material`。PyCharm 需要正确识别源码搜索路径，并通过所选解释器的环境信息解析包与函数引用。

- **Sources Root**：告诉 IDE 将 `src` 作为代码解析的源码目录。
- **解释器重新加载**：让 IDE 重新获取解释器和已安装包的信息，可能恢复此前未更新的静态分析与索引。
- **`uv pip install -e .`**：把本地项目以 editable 模式安装到指定虚拟环境，主要保证该环境的 Python 运行时能导入包；不等同于强制 PyCharm 立即刷新代码索引。

本次确认的是上述组合操作**有效**，不能仅凭结果断定单一根因是索引缓存、解释器配置还是路径识别。

## 4. 下次快速排查顺序

1. 确认 PyCharm 打开的是包含 `pyproject.toml` 的**仓库根目录**。
2. 将 `src` 标记为 **Sources Root**。
3. 确认项目解释器指向当前项目的 `.venv`，然后**重新加载解释器**并等待索引。
4. 若仍失败，再验证 `uv pip install -e .` 是否安装到了 **PyCharm 所使用的同一个解释器**。
5. 最后才考虑 PyCharm 的 Repair IDE / Invalidate Caches；不要一开始就删除 `.idea` 或重建项目。

可选命令（在项目根目录、Windows PowerShell 中）：

```powershell
uv pip install -e . --python .\.venv\Scripts\python.exe
.\.venv\Scripts\python.exe -c "from cst_material.geometry import box_volume; print(box_volume.__module__)"
```

> 这组命令是后续诊断参考，**不是本次最终恢复跳转的直接步骤**；本次真正恢复是在重新加载 PyCharm 解释器后。

## 5. 验证标准

- 本地包导入不再出现无法解析的波浪线。
- Ctrl + 点击 `BoundingBox` 可跳转到 `src/cst_material/domain.py`。
- Ctrl + 点击 `box_volume` 或 `center` 可跳转到 `src/cst_material/geometry.py`。

## 6. 经验总结

**Python 包可安装、可运行，不代表 IDE 的静态索引已同步。** 对 `src layout` 工程，优先检查源码根目录、项目解释器和解释器索引，而不是修改原本正确的包导入语句或重构代码目录。
