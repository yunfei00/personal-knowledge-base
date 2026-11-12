---
id: fix-powershell-input-automatic-variable-001
title: PowerShell $input 自动变量冲突：文件路径参数传递失败
category: troubleshooting
tags: [PowerShell, Windows, 自动变量, input, inputFile, CST, Python, CLI]
status: verified
created: "2026-11-12"
updated: "2026-11-12"
platform: "Windows / PowerShell"
source_project: "yunfei00/cst-material-automation"
source_url: "https://github.com/yunfei00/cst-material-automation/blob/feat/cst-p3d-large-neighbors/docs/p3d-large-neighbors-verification.md"
sensitivity: public
---

# PowerShell $input 自动变量冲突：文件路径参数传递失败

> **实际解决结果（2026-11-12）：** 在运行 CST P3-D 离线空间分析的 PowerShell 命令时，示例误把 `$input` 当作普通文件路径变量。用户已经改用 **`$inputFile`**，再次运行没有报错，确认已解决。无需重复验证已经成功的执行流程。

## 问题现象

原始示例使用：

```powershell
$input = 'local-data\P3C_Lock_Diagnostics.json'
python -m cst_material.cli spatial-offline --report-in $input --report 'local-data\P3D_Neighbors_Report.json'
```

在 PowerShell 中不能可靠地按普通用户变量传递 `$input`，导致路径参数与预期不符。

**根因：** `$input` 是 PowerShell 的**自动变量**，用于访问传递给函数或脚本块的管道输入对象/枚举器，并非普通的业务变量名。代码示例不应使用它存储文件路径。

## 已验证解决方案

改用**非保留、语义明确**的路径变量，如 `$inputFile`、`$outputFile`：

```powershell
$inputFile = 'local-data\P3C_Lock_Diagnostics.json'
$outputFile = 'local-data\P3D_Neighbors_Report.json'

python -m cst_material.cli spatial-offline --report-in $inputFile --report $outputFile --neighbors 5 --batch-size 32
```

实际反馈：**用户使用的是 `$inputFile`，运行已正常；不是 `$interface`。**

## 复用原则

- PowerShell 中不要使用 `$input` 作为业务变量；相关内建/自动变量也应避免覆盖，具体可参考 PowerShell 关于自动变量（about_Automatic_Variables）的官方文档。
- 文件路径变量采用 `$inputFile`、`$sourcePath`、`$reportPath`、`$outputFile` 等名称；给外部 CLI 传路径前，建议先用 `Test-Path -LiteralPath` 检查。
- 对大体积 JSON 只打印关键摘要，不要把几十万行全文打印到终端。
- 本次用户提到**接近 25 万行的 JSON 和最后 `labels.status=SKIPPED` 是先前旧报告的现象**；不要把旧报告的状态直接解释成当前 P3-D 执行失败，也不要混淆 `labels.status` 与 `neighbors.status`。

## 相关项目

- [CST P3-D 分批邻近分析验证说明](https://github.com/yunfei00/cst-material-automation/blob/feat/cst-p3d-large-neighbors/docs/p3d-large-neighbors-verification.md)
- [PowerShell 自动变量（Microsoft 官方文档）](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_automatic_variables)
