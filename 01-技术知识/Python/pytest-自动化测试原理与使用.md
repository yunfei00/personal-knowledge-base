---
id: tech-python-pytest-001
title: "pytest 自动化测试：原理、用法与 CST 项目实例"
category: technical
tags: [Python, pytest, 自动化测试, 单元测试, uv, CI]
status: maintained
created: "2026-11-11"
updated: "2026-11-11"
platform: "Windows / Python 3.11"
source_project: "cst-material-automation"
source_url: "https://github.com/yunfei00/cst-material-automation"
sensitivity: public
---

# pytest 自动化测试：原理、用法与 CST 项目实例

## 是什么

**pytest 是 Python 的自动化测试工具**。它会自动找到符合约定的测试函数，执行这些函数，检查实际结果是否符合预期，并汇总通过、失败和跳过的数量。

举例：写一个计算长方体体积的函数。每次修改代码后，不需要手工打开程序、输入长宽高并计算对错，只要运行 pytest，就能自动检查结果是否正确。

注意区分：

- **Python**：执行程序的语言和运行环境。
- **uv**：管理 Python 版本、依赖和虚拟环境，并可在项目环境中运行命令。
- **pytest**：真正负责发现、执行和汇总测试的框架。
- **Ruff**：检查 Python 代码风格、潜在错误等；不等于功能测试。
- **GitHub Actions / CI**：在提交代码后自动运行上述检查的平台；CI 不是 pytest 的替代品。

## 原理

一次 pytest 测试通常经过四步：

1. **发现测试**：按默认规则寻找 `test_*.py`、`*_test.py` 等文件，以及其中的 `test_*` 函数。
2. **执行测试**：调用测试函数，运行其中的 Python 代码。
3. **检查断言**：`assert 实际值 == 预期值` 为真则继续；为假则抛出 `AssertionError`，该测试失败。
4. **汇总结果**：输出 `passed`、`failed`、`skipped` 等统计，并通过退出码告诉命令行或 CI 是否成功。

例如：

```python
# geometry.py
def volume(length, width, height):
    return length * width * height

# test_geometry.py
from geometry import volume

def test_volume():
    assert volume(2, 3, 4) == 24

def test_zero_height():
    assert volume(2, 3, 0) == 0
```

这里的 `24` 和 `0` 是我们事先定义的**正确答案**，不是 pytest 自动推导出来的。测试是否有价值，取决于预期结果是否正确、用例是否覆盖重要情况。

如果程序错误地返回 `25`，第一个测试就会失败。**测试全部通过，只表示现有测试覆盖的行为符合预期，不代表所有功能都没有问题。**

## 怎么用

在已经配置 `uv` 和 pytest 依赖的项目根目录中：

| 命令 | 作用 |
| --- | --- |
| `uv run pytest` | 执行当前项目收集到的全部测试 |
| `uv run pytest -v` | 显示每个测试的详细结果 |
| `uv run pytest tests/test_geometry.py` | 只运行指定文件 |
| `uv run pytest -k volume` | 只运行名称包含 `volume` 的测试 |
| `uv run pytest -m "not cst_real"` | 排除标记为 `cst_real` 的测试 |
| `uv run pytest -m "cst_real"` | 只执行标记为 `cst_real` 的测试 |
| `uv run ruff check .` | 检查代码规范；不是运行测试 |

**重点理解命令 `uv run pytest -m "not cst_real"`：**

- `uv run`：在 uv 管理的项目环境中执行后面的命令。
- `pytest`：启动测试框架。
- `-m`：按照 **marker（标记）表达式**筛选测试。
- `"not cst_real"`：逻辑取反，意思是**不执行被标记为 `cst_real` 的测试**。

`cst_real` 不是 CST 自带命令，而是项目开发者自定义的 pytest 标记。应在项目配置中注册，避免未知标记警告。

## 示例：CST 项目为什么区分两种测试

`cst-material-automation` 需要区分：

**A. 通用单元测试**：验证几何计算、JSON 生成、状态判断等纯 Python 功能。GitHub CI 没有 CST，也能执行。

**B. 真实 CST 集成测试**：需要在安装了 CST 2025 SP1 的 Windows 电脑上，连接真实软件、读取 Solid、修改测试工程副本并验证保存结果。这些测试以后应加上 `@pytest.mark.cst_real`。

```python
import pytest

def test_box_volume():
    assert 2 * 3 * 4 == 24

@pytest.mark.cst_real
def test_cst_connection():
    # 示例占位：未来需调用已经确认存在的真实 CST 接口。
    # 不应仅写 pass 就宣称真实连接通过。
    pytest.skip("真实 CST 连接测试尚未实现")
```

当运行 `uv run pytest -m "not cst_real"` 时，第一项会执行，第二项因为标记被排除，**不会尝试连接 CST**。

即使结果是 `10 passed`，也只能证明这 10 项已执行的测试通过；不能说明 CST 材料赋值已成功。

### pytest、skip、xfail 和未运行的区别

- `PASSED`：测试确实执行，且所有断言满足。
- `FAILED`：测试执行了，但断言失败或发生未处理异常。
- `SKIPPED`：测试被显式跳过。
- `XFAIL`：标记为预期失败，通常用于已知缺陷。
- **Deselected（未选中）**：被 `-m`、`-k` 等筛选条件排除，根本没有执行。
- **NOT_RUN**：本项目验收报告中的业务状态，不是 pytest 的标准测试结果；表示某项验收还没有执行。

## 测试结果如何理解

常见输出：

```text
================== 8 passed, 2 deselected in 0.42s ==================
```

含义：8 项真正执行并通过，另外 2 项被筛选排除。**不是 10 项全部通过**。

若出现：

```text
FAILED tests/test_geometry.py::test_volume
AssertionError: assert 25 == 24
```

表示该测试的实际结果与预期不一致，应检查代码或测试预期，而不是直接删除测试让 CI 变绿。

## 常见疑问

**1. pytest 会自动替我写测试吗？**

不会。它负责运行测试，测试函数和正确的预期结果需要开发者编写。

**2. 为什么每次修改代码都要测试？**

新修改可能影响旧功能。重复执行同一组测试叫做**回归测试**，目的是尽早发现被改坏的功能。

**3. CI 绿了，为什么还要在公司电脑测试？**

GitHub Actions 通常没有安装 CST 2025 SP1，无法验证真实 CST API、授权、工程文件和保存回读。CI 通过只是必要检查之一。

**4. 测试失败是否说明程序完全不能用？**

不一定。失败可能是程序缺陷、测试预期错误、依赖缺失或环境问题。需要根据失败日志定位原因。

**5. pytest 与 unittest 有什么区别？**

两者都是 Python 测试框架。unittest 属于 Python 标准库；pytest 是第三方框架，支持简洁的 `assert`、fixture、参数化和标记筛选，也能运行很多 unittest 风格测试。

**6. fixture 是什么？**

fixture 是 pytest 提供的测试前置准备和资源清理机制。例如创建临时文件夹、准备测试数据、测试结束后释放资源。通常通过 `@pytest.fixture` 定义，由测试函数按参数名使用。

## 注意事项

- 真实 CST 测试必须在获授权的本地环境运行，避免修改原始工程。
- `pytest -m "not cst_real"` **不是安全沙箱**；未正确标记的测试仍会执行。运行前应审查测试内容和标记。
- 不要用仅包含 `pass` 的空测试充当有效验收。
- 不要把公司工程文件、材料数据、客户信息或原始日志提交到公开仓库。
- 对真实集成测试采用单独标记、受控副本、回读检查和明确的 PASS/FAIL/BLOCKED/NOT_RUN 记录。

## 参考资料

- [pytest 官方文档](https://docs.pytest.org/en/stable/)
- [pytest 标记筛选](https://docs.pytest.org/en/stable/example/markers.html)
- [uv 官方文档](https://docs.astral.sh/uv/)
- [CST Material Automation 仓库](https://github.com/yunfei00/cst-material-automation)
