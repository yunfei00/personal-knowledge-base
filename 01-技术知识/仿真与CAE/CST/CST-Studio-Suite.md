---
id: tech-cst-intro-001
title: CST Studio Suite 入门：Solid、Component 与 Material
category: technical
tags: [CST, CST Studio Suite, Solid, Component, Material, MTD, 材料库, SAT, 三维建模, 电磁仿真]
status: draft
created: "2026-11-11"
updated: "2026-11-12"
platform: "CST Studio Suite 2025 / Windows"
source_project: ""
source_url: "https://www.3ds.com/products/simulia/cst-studio-suite"
sensitivity: public
---

# CST Studio Suite 入门：Solid、Component 与 Material

> 本文是持续维护的 CST 入门笔记。先解释最常见的概念，后续遇到具体问题时继续补充。以下以 **CST Studio Suite 2025** 为主要使用背景，不涉及公司工程原始数据。

## 1. CST 是什么？

**CST Studio Suite** 是 SIMULIA（达索系统）旗下的电磁场仿真软件套件，可用于天线、射频、微波、电磁兼容（EMC）、电磁干扰（EMI）等领域的建模与分析。

一般工作过程：

1. 创建或导入几何结构。
2. 检查几何体的组织关系和材料。
3. 选择适当的求解器、边界条件、激励源等。
4. 划分网格并运行仿真。
5. 查看场分布、S 参数或其他结果。

**注意：** 导入几何模型不等于已经完成仿真模型配置。

## 2. Solid 是什么？

**Solid（实体）** 是 CST 中表示一个三维几何实体的对象。它具有具体形状、尺寸、位置等几何属性，通常还关联一个材料。

例如，一部手机的简化结构可能包括：

- 金属边框：一个或多个 Solid。
- 塑料支架：一个或多个 Solid。
- PCB 基板：一个或多个 Solid。
- 屏蔽罩：一个或多个 Solid。

可以把 Solid 理解成：**“模型里的一块三维实体”**。

一个物理零件不一定恰好对应一个 Solid：CAD 导入、布尔运算、结构拆分都可能使一个零件对应多个实体，或使多个设计特征组合成一个实体。不要仅凭 Solid 数量判断物料数量。

## 3. Component 是什么？

**Component（组件）** 用于在 CST 的模型树中组织几何实体，便于管理、查找、显示或隐藏。

例如：

```text
Components
├── Housing
│   ├── solid1
│   └── solid2
├── PCB
│   ├── substrate
│   └── shield
└── Antenna
    └── radiator
```

上面名称只是教学示例，并非 CST 自动识别的固定名称。

**区别：** Component 是组织层级；Solid 是具体几何实体。不要把 Component 名称直接当成材料或零件真值。

## 4. Material 是什么？

**Material（材料）** 用来描述实体在仿真中的物理属性。电磁仿真常涉及：

- 相对介电常数（Relative Permittivity，εr）。
- 相对磁导率（Relative Permeability，μr）。
- 电导率（Conductivity）。
- 介质损耗或其他频率相关属性。

常见材料概念包括 **PEC（理想电导体）**、铜、空气、塑料和介质基板。真实工程中的材料参数需要依据工程资料、测量或已确认的材料库，不能只凭外观推断。

## 5. Solid、Component、Material 的关系

| 概念 | 回答的问题 | 例子 |
| --- | --- | --- |
| Solid | 这是什么形状、在哪里？ | 一块长方体 PCB 基板 |
| Component | 这些实体如何分组管理？ | PCB 组件 |
| Material | 这个实体具有什么物理属性？ | 某种介质材料 |

```text
Component（组织多个实体）
    ├── Solid A ──→ Material X
    ├── Solid B ──→ Material Y
    └── Solid C ──→ Material X
```

多个 Solid 可以引用同一材料定义；Solid 的几何形状和 Material 的物理属性是不同层面的信息。

## 6. SAT / ACIS 导入后会发生什么？

**SAT** 是 ACIS 几何数据的常见交换格式之一。导入 CST 后，主要得到可供 CST 使用的几何对象及其结构信息；是否带有可靠的材料定义，取决于源文件、导出方式和导入结果。

对于**只有结构、没有可靠材料信息**的输入，不能认为 CST 已经知道每个 Solid 应该使用什么材料。

建议首先检查：

1. 是否成功导入全部预期几何体。
2. Solid 的数量、尺寸、位置和层级是否合理。
3. 是否出现实体丢失、重叠或不合理的几何拆分。
4. 各 Solid 当前关联了什么材料，是否只是默认值。
5. 保存 CST 工程并重新打开后，几何和材料是否仍然一致。

## 7. 自动识别和赋材料时需要理解的区别

在结构自动赋材料场景中，应将两个任务分开：

- **几何识别：** 根据尺寸、形状、厚度、位置、接触关系等判断一个 Solid 可能承担的结构功能。
- **材料赋值：** 根据已确认的工程材料知识与匹配规则，为该 Solid 指定 CST 中可用的材料定义。

例如，某个薄片状 Solid 可能是金属片，也可能是介质片；**仅凭薄这一特征无法确定材料**。不确定时应保留待确认状态，而不是强行赋值。

对于长期可复用的自动化工具，不应把文件名、Solid 名称或 Component 名称作为唯一识别依据。

## 8. 常见术语速查

| 术语 | 简单解释 |
| --- | --- |
| Project（工程） | 保存模型、设置及相关仿真内容的工作单元 |
| Model Tree（模型树） | 管理几何、材料等对象的树状界面 |
| Solid（实体） | 三维几何对象 |
| Component（组件） | Solid 的组织分组 |
| Material（材料） | 电磁或其他仿真所需的物理属性定义 |
| SAT / ACIS | 几何数据格式及相关几何建模技术 |
| Boolean（布尔运算） | 对几何体执行合并、相减、相交等操作 |
| Mesh（网格） | 求解器离散化模型所用的计算单元 |
| Solver（求解器） | 计算电磁问题的数值方法或模块 |
| Boundary Condition（边界条件） | 规定计算区域边界的物理处理方式 |

## 9. 在 CST 2025 的哪里查看 Solid 列表？

打开 CST 左侧 **Navigation Tree（导航树）→ Components**，逐层展开 Component，即可看到下属的三维 Solid；它们通常按组件分组，而不是在顶层提供一个单独的 `Solids` 列表。

- 单击一个 Solid，可以定位对应的三维几何体。
- 选中 Solid，通过 **Modeling → Materials → New/Edit → Assign Material and Color** 查看、修改当前材料。
- 左侧 **Materials** 节点列出**当前工程内**已经定义/加载的材料；它和外部 Material Library 对话框不是同一个概念。
- Component 不等于 Solid；不能将导航树下所有文件夹或叶子项不加判断地视为实体。

## 10. 自定义 MTD 材料库：容易混淆的三个操作

1. **注册外部材料库**：通过 **File → Manage Libraries → Library paths** 添加本地库目录，使外部 `.mtd` 材料在库中可见；已实际配置成功。
2. **Load 到当前工程**：通过 **Modeling → Materials → Material Library → Load from Material Library** 选择材料并加载到当前工程的 `Materials`。不需要指定任何 Solid。
3. **给 Solid 指定材料**：通过 **Assign Material and Color** 或 VBA `Solid.ChangeMaterial`，把工程内已存在的材料关联到目标 Solid。

**特别注意：** 在外部库中看到 100 个 MTD，不代表该工程的 `Materials` 已包含这 100 个材料；而工程内部已经 Load 并保存的材料通常可以随工程被其他电脑读取，但涉及特殊外部数据、更新材料参数和仿真复现时仍要单独检查。

完整说明（含外部库目录组织、批量加载与批量赋值的区别、跨电脑移交、材料更新和验证步骤）：**[CST 2025 外部 MTD 材料库：配置、加载、赋值与工程移交](CST-外部MTD材料库与工程材料管理.md)**。

## 11. CST Python 和 VBA 怎样配合？

- Python：`cst.interface.DesignEnvironment()`、`env.open_project(...)`、`project.model3d.get_tree_items()`，用于打开项目、读取工程树。
- VBA Solid：`Solid.GetNumberOfShapes`、`Solid.GetNameOfShapeFromIndex`、`Solid.GetMaterialNameForShape`、`Solid.ChangeMaterial`，用于准确识别和赋材。
- VBA Material：`Material.GetNumberOfMaterials`、`Material.GetNameOfMaterialFromIndex`、`Material.Exists`、`Material.Create`，用于工程内部材料的检查与创建。

这些接口名称已在 CST 2025 官方离线帮助中确认；但具体自动化工具是否完整运行通过，须以真实 CST 工程的 Windows 实机测试为准，不能以 GitHub CI 的模拟测试代替。

当前代码与验证文档：[cst-material-automation P1/P2](https://github.com/yunfei00/cst-material-automation/blob/feat/cst-p0-probe/docs/p1-p2-verification.md)。

## 12. 软件安装与维护：CST 2025 SUP 补丁安装（已验证）

**记录日期：** 2026-11-11  
**环境：** CST Studio Suite 2025 / Windows  
**补丁格式：** `.sup`（Service Pack）  
**实际结果：** SUP 文件导入成功，补丁安装成功。具体补丁编号及更新后的版本号尚未记录。

### 安装步骤

1. 保存工程并关闭 CST Studio Suite 2025。
2. 在 Windows 开始菜单中打开对应版本的 **CST Update Manager**。
3. 点击 **Import**，选择本地 `.sup` 补丁文件。
4. 等待导入完成，在更新列表中选择该补丁。
5. 点击 **Install**，等待安装结束。
6. 重新打开 CST，建议通过 **Help / About** 核对版本，并验证常用功能。

### 注意事项

- 确认 SUP 补丁与 CST 2025 主版本兼容。
- 不要直接把 SUP 文件复制到安装目录。
- 如遇权限不足，可尝试以管理员身份运行更新管理器。
- 本次仅确认“导入成功、安装成功”；软件功能回归、补丁编号及版本号未核实，不标记为已验证。

**快速记忆：** `CST Update Manager → Import → 选择 .sup → Install`

**检索关键词：** CST、2025、SUP、Service Pack、补丁、Update Manager。

## 13. CST Python 接口包的离线安装（待本机验证）

用于理解如下命令（`path` 需要替换为实际安装包目录）：

```powershell
python -m pip install --no-index --find-links "path" cst-studio-suite-link
python -m pip show cst-studio-suite-link
```

- `--no-index`：不访问 Python 包索引。
- `--find-links`：在指定目录查找可用的包及其依赖。
- `cst-studio-suite-link`：待安装的 Python 分发包名。
- 先确认 CST 2025 安装目录中确实提供兼容的包文件；不同版本、安装方式的目录可能不同，不应直接套用未经核实的路径。
- `pip show` 只能确认包的安装信息，**不能证明 CST 自动化接口可用**。后续还应使用同一个 Python 环境执行实际的 CST 导入、连接、工程保存及回读测试。
- 本节只记录命令含义和建议验证步骤，**尚未在当前 CST 2025 环境验证安装成功**。

通用参数解释：[pip 安装命令与离线安装速查](../../Python/pip-安装与离线安装.md)。

## 14. 后续待补充主题

- 如何查看 Solid 的尺寸、体积、包围盒及相对位置。
- 特殊几何、材料属性和仿真求解设置的排错。
- SAT 导入后的几何修复与检查（仅作为 CST 通用知识，当前自动化项目不需要导入 SAT）。
- 工程在另一台无外部 MTD 库的电脑上打开、回读关键材料参数、仿真复现的真实验收记录。

## 参考资料

- [SIMULIA CST Studio Suite 产品介绍](https://www.3ds.com/products/simulia/cst-studio-suite)
- CST Studio Suite 2025 离线帮助：`mergedProjects/3D/common_general/`、`mergedProjects/VBA_3D/`、`userinterface/elements/backstage.htm`（帮助文档仅供本机核对，不复制厂商帮助到公开仓库）
- [CST 2025 外部 MTD 材料库与工程材料管理](CST-外部MTD材料库与工程材料管理.md)
