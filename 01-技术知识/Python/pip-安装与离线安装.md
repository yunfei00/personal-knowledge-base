---
id: tech-python-pip-offline-001
title: pip 安装命令与离线安装速查
category: technical
tags: [Python, pip, 离线安装, no-index, find-links, wheel]
status: maintained
created: "2026-11-11"
updated: "2026-11-11"
platform: "Windows / Linux / Python"
source_project: ""
source_url: "https://pip.pypa.io/en/stable/cli/pip_install/"
sensitivity: public
---

# pip 安装命令与离线安装速查

## 1. pip 是什么？

`pip` 是 Python 的包安装工具。安装目标是当前调用的 Python 环境；机器上有多个 Python 或虚拟环境时，`pip` 可能指向不同解释器，建议使用 `python -m pip` 明确目标。

## 2. 命令逐项解释

```powershell
python -m pip install --no-index --find-links "path" cst-studio-suite-link
```

| 部分 | 含义 |
| --- | --- |
| `python -m pip` | 使用当前 `python` 对应环境的 pip |
| `install` | 安装指定包 |
| `--no-index` | 不从 PyPI 或配置的其他包索引查找软件包 |
| `--find-links "path"` | 从指定本地目录或支持的链接页面查找安装候选文件 |
| `cst-studio-suite-link` | 要安装的包名称 |

`"path"` 只是占位符，必须替换为**实际存在且含有匹配安装包**的路径。此命令不会自动下载缺失的依赖；如果本地没有兼容版本或依赖包，安装会失败。

## 3. Windows 本地目录示例

以下目录只是示例，**不是已确认的 CST 安装路径**：

```powershell
python -m pip install --no-index --find-links "D:\\offline-wheels" cst-studio-suite-link
python -m pip show cst-studio-suite-link
```

离线目录一般放置 `.whl` 或源码分发包。安装前可用资源管理器确认目录内确实有目标包文件；包文件还需要与 Python 版本、操作系统及架构兼容。

## 4. 与普通安装的区别

- `python -m pip install 包名`：通常会从配置的包索引查找、下载。
- `python -m pip install --no-index --find-links "目录" 包名`：仅从指定位置等本地可用来源寻找，不访问包索引。
- `python -m pip install "完整路径\\包文件.whl"`：直接安装指定的 wheel 文件；依赖仍需满足，必要时可配合离线选项。

## 5. 验证与常见错误

- `No matching distribution found`：检查目录、文件名、Python 版本、平台架构和包名。
- 依赖找不到：把所需依赖的兼容安装包一并放进离线目录。
- 安装成功却无法导入：检查运行脚本的 Python 与安装使用的 Python 是否相同；分发包名称不一定等于 `import` 名称。
- 只想查看安装信息：`python -m pip show 包名`。

## 6. CST 相关案例

CST Python 接口包的具体安装目录与安装后的连接验证，见 [CST Studio Suite 入门](../仿真与CAE/CST/CST-Studio-Suite.md)。本篇只解释通用 pip 原理，不把示例路径当成已验证环境。
