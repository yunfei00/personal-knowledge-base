---
id: tech-cst-mtd-material-library-001
title: CST 2025 外部 MTD 材料库：配置、加载、赋值与工程移交
category: technical
tags: [CST, CST Studio Suite, MTD, 材料库, Manage Libraries, Material, Solid, VBA, 工程移交]
status: draft
created: "2026-11-12"
updated: "2026-11-12"
platform: "CST Studio Suite 2025 / Windows"
source_project: ""
source_url: ""
sensitivity: public
---

# CST 2025 外部 MTD 材料库：配置、加载、赋值与工程移交

> **一句话记忆：** 外部材料库负责“提供可选择的材料”，工程内的 `Materials` 负责“保存当前工程的材料定义”，`Solid` 的材料属性负责“把材料分配给哪个几何实体”。三件事不能混为一谈。
>
> **状态说明：** 外部 MTD 库路径配置并能显示自定义材料，已经在 CST 2025 实际使用中确认；“脱离外部库后跨电脑完整打开、参数与仿真行为不变”尚未完成独立验收，不能写为百分之百保证。本文只记录通用方法，不包含公司的实际工程和材料参数。

## 1. 三个层次：材料库、工程材料、Solid 赋值

```text
外部自定义材料库（多个 .mtd 文件）
    │
    └── 在 CST 材料库界面中显示、选中材料
              │
              └── Load from Material Library
                         │
                         ▼
              当前 .cst 工程的 Materials
              （已加载的材料定义及属性）
                         │
                         └── Assign Material and Color
                              / Solid.ChangeMaterial
                                      │
                                      ▼
                              Components → 某个 Solid
```

| 当前状态 | 含义 | 复制 `.cst` 到另一台没有自定义库的电脑 |
| --- | --- | --- |
| 只在外部 Material Library 列表中可见 | 本机已找到库文件，但材料未必加载到项目 | **不能**认为该库所有材料已包含在工程中 |
| 已通过 **Load** 加入当前工程 `Materials` | 工程拥有对应材料定义 | 保存后**通常**可随工程打开；需做移交测试 |
| Solid 已指定该工程内材料 | 几何体关联工程的材料定义 | 通常可保留关系；仍应核对材料参数与外部依赖 |

**关键误区：** 注册一个装有几百个 MTD 的目录，只是让 CST *看到*这些材料；不等于已将几百种材料都导入当前工程。给 Solid 赋材也不是加载材料定义的前提。

## 2. 外部自定义 MTD 文件怎么用？

### 2.1 整理材料文件

材料可以存放在 CST 安装目录之外，例如：

```text
D:\CST_Custom_Library\
└── Materials\
    ├── Material_A.mtd
    ├── Material_B.mtd
    └── Material_C.mtd
```

这是**便于管理的示例布局**，不是要求所有 CST 版本都必须使用某个固定磁盘路径。外部库的目录组织应以本机已经成功识别的配置为准，不要随意覆盖或移动正在使用的材料文件。

### 2.2 把外部路径注册给 CST

CST 2025 可通过 **File → Manage Libraries → Library paths** 管理外部库路径；在对应列表新增已准备好的材料库根目录并保存。已实际验证：添加路径后，库中的自定义材料能够在材料库选择界面显示。

官方帮助还说明：

- 库路径**有优先级**：列表上方的目录优先查找；同名资源时需要特别小心加载来源。
- 需要将新数据写入库时，优先使用列表最上方的可写目录；因此应先明确你要管理的自定义库位置。
- 路径属于**本机 CST 配置**，不要因为工程在一台电脑上能找到外部库，就假设其他电脑拥有相同配置。

### 2.3 把材料真正加载到当前 CST 工程

在现有 CST 工程中使用：

**Modeling → Materials → Material Library → Load from Material Library**

在库中选中所需材料后点击 **Load**。根据 CST 2025 帮助，Load 的结果是：将**选中的材料**加载到**当前工程**。随后在左侧 Navigation Tree 的 `Materials` 节点查看它是否已经出现。

如果 Solid 还没有用到该材料，也可以先 Load；**加载材料不要求指定 Solid**。

### 2.4 自己创建的材料如何放回库？

CST 还提供 **Modeling → Materials → Material Library → Add to Material Library**，用于把当前工程中的材料添加回材料库。它和 Load 的方向相反：

- **Load**：材料库 → 当前工程。
- **Add**：当前工程 → 材料库。

## 3. 能否一次处理全部 MTD 材料？

必须区分以下两种“批量”：

1. **一次注册整个外部库：可以。** 将多个 MTD 放在能够被 CST 识别的材料库目录中，配置一次路径，即可让这些材料成为库中的候选资源。
2. **一次把库中全部材料 Load 到某个工程：不能仅凭路径注册推断已经完成。** CST 2025 的 `Load from Material Library` 帮助描述的是“选定材料 → Load”，没有据此确认 UI 一键加载整个库的功能。

自动化思路也需要分开：

- **批量创建材料定义：** 如果已知名称和物性参数，可用 CST VBA 的 `Material.Name`、`Material.Type`、`Material.Create` 等逐个创建；**并不意味着可以直接把任意 MTD 文件交给 `Material.Create` 自动解析**。
- **批量赋材料给 Solid：** 当材料已经存在于当前工程，可用 `Solid.ChangeMaterial` 循环赋值；**不等于把外部 MTD 库导入工程**。
- **直接批量读取多个 MTD 并 Load：** 需要另外核对可用的专用接口或实际 UI 自动化能力，在确认之前不写成“已实现”。

## 4. 如果复制 CST 工程到其他电脑，外部 MTD 库不在了会怎样？

### 4.1 通常的情况

如果材料已经**加载到工程的 Materials 中**，并保存了 CST 工程，那么移交工程时，接收方通常仍能读取这些**已保存的材料定义**及 Solid 的材料关联；不需要为了查看工程里已经拥有的材料，重新从原电脑外部库 Load 一遍。

但应区分：

- **材料是否显示**：工程打开后 `Materials` 里能看到名称。
- **材料参数是否正确**：介电常数、电导率、损耗、频率相关数据等是否一致。
- **仿真能否完整复现**：材料模型、外部数据、工程关联资源、版本兼容性与求解设置是否满足条件。
- **能否更新或从库重新 Load**：如果接收电脑没有外部库，将无法按原路径重新检索这些外部材料。

**不要把“工程能打开”当成“材料参数和仿真结果完全无外部依赖”。**

### 4.2 最稳妥的离线移交验证

在**测试工程副本**上执行：

1. 通过材料库 **Load** 一种自定义 MTD 材料到当前工程 `Materials`。
2. 给一个测试 Solid 指定该材料，保存并关闭 CST。
3. 临时**移除或禁用外部 Library path**（不要删除原始 MTD 文件），再启动 CST。
4. 重新打开保存的工程，检查 `Materials` 节点、Solid 关联的材料，以及关键参数是否存在、数值是否正确。
5. 有条件时，再在**从未配置过该库的电脑**上测试，并检查模型重建和相关仿真设置。

结果通过后，才能针对该工程和所使用的材料模型记录“脱离外部库可移交”。如果工程还依赖其他外部数据、结果目录或关联文件，应一并检查；CST 的 **Pack Project** 可作为研究项目打包的功能入口，但不要默认它会修复所有外部依赖。

### 4.3 注意“从材料库更新属性”

CST 2025 在 Navigation Tree 的 `Materials` 上提供 **Update All Properties from Material Library**，也可在单个材料上使用 **Update Properties from Material Library**。

这说明项目内材料**可以依据库中同名材料更新属性**；因此跨电脑移交后若主动执行更新，可能使用接收电脑的材料库版本，造成参数变化。复现实验前应保留项目材料参数快照，必要时记录材料版本或来源。

## 5. Solid 在哪里看？怎么知道每个 Solid 用的是什么材料？

打开 CST 左侧 **Navigation Tree → Components**，展开具体 Component，下面的三维实体节点就是 Solid，例如：

```text
Components
├── Housing
│   ├── solid1
│   └── solid2
└── PCB
    ├── substrate
    └── shield

Materials
├── Copper
└── Custom_Plastic
```

图中名称均为示例。**CST 不一定存在一个独立叫 `Solids` 的顶层列表**；Solid 是按 Component 组织的。

查看或修改指定 Solid 的材料：选中该实体，打开 **Modeling → Materials → New/Edit → Assign Material and Color**（也可从所选实体的相关操作入口进入），在对话框查看当前 Material。该界面允许从项目已有材料中选择，也提供 **New Material** 和 **Load from Material Library** 的入口。

- `Component`：分组/组织层级，不等于一个具体零件。
- `Solid`：实际三维几何实体。
- `Material`：材料定义，可被多个 Solid 使用。

## 6. Python / VBA 自动化能做什么？

Python 接口用于连接项目、操作模型历史和保存工程；涉及三维 Solid 及材料对象时，可以通过 CST 2025 VBA API。

| 官方 API | 用途 | 说明 |
| --- | --- | --- |
| `model3d.get_tree_items()`（Python） | 读取工程导航树 | 树的叶子节点不应未经核对全部当作 Solid |
| `Solid.GetNumberOfShapes` | 获取当前 Solid 数量 | 可用于独立核对工程树 |
| `Solid.GetNameOfShapeFromIndex` | 按索引获取 Solid 名称 | 用于枚举真实实体 |
| `Solid.GetMaterialNameForShape` | 查询某 Solid 的材料 | 用于赋值前后核对 |
| `Material.GetNumberOfMaterials` / `GetNameOfMaterialFromIndex` | 枚举工程内材料 | 官方文档写明材料索引从 0 开始 |
| `Material.Exists` | 验证材料是否存在 | 不存在则不应直接赋值 |
| `Material.Create` | 按已设置的物性参数创建材料 | 不代表读取外部 MTD 文件 |
| `Solid.ChangeMaterial` | 将**工程中已存在**的材料赋予 Solid | 材料必须先存在 |

**当前自动化项目的边界：** `cst-material-automation` 的 P1/P2 采用“已有 CST 工程 → 读取结构和材料 → 修改副本 → 保存和回读”；不把外部 MTD 批量导入算作已实现功能。GitHub 单元测试通过不等于 CST 2025 SP1 实机通过。

代码与业务验收说明：[cst-material-automation / P1-P2 验证文档](https://github.com/yunfei00/cst-material-automation/blob/feat/cst-p0-probe/docs/p1-p2-verification.md)。

## 7. 常见问题速查

| 问题 | 结论 |
| --- | --- |
| 能否直接用外部 MTD？ | 可以将外部材料目录注册为 CST 库，先让材料在库中可选 |
| MTD 必须放 CST 安装目录吗？ | 不必，已可通过外部 Library paths 使用 |
| 材料必须先指定 Solid 才能导入吗？ | 不需要，先 Load 到工程，再按需赋给 Solid |
| 材料库里显示了 100 种，工程就含有 100 种吗？ | 不一定，只有实际 Load / 创建到工程里的材料才算工程内已有定义 |
| 能否一次注册多个 MTD？ | 可以把目录作为外部库加入；**不等同于一次性全部 Load 到工程** |
| 换电脑没有外部库还能看 Solid 材料吗？ | 已 Load、保存的工程通常可以；关键物性、特殊外部依赖仍要验证 |
| 材料库修改后，旧工程材料参数自动改变吗？ | 不能假定；CST 提供从库中主动更新工程材料属性的操作 |
| Solid 在哪里？ | 左侧 Navigation Tree → Components → 对应 Component 下 |
| 如何安全批量赋材？ | 先检查工程内存在目标 Solid 和材料，再在副本上修改、保存、重新打开核对 |

## 8. 来源、验证情况和后续

**CST 2025 离线帮助来源（用户提供的 Online Help，未上传至公开知识库）：**

- `userinterface/elements/backstage.htm`：Manage Libraries 与 Library paths 的优先级。
- `mergedProjects/3D/common_general/common_general_load_from_material_library.htm`：Load 到项目。
- `mergedProjects/3D/common_general/common_general_add_to_material_library.htm`：Add 到库。
- `mergedProjects/3D/common_general/common_general_update_properties_from_material_library.htm`：项目材料从库更新。
- `mergedProjects/3D/common_struct/common_struct_change_material.htm`：Assign Material and Color。
- `mergedProjects/VBA_3D/common_vbasolido/common_vbasolido_solid_object.htm`：Solid 查询与材料赋值。
- `mergedProjects/VBA_3D/special_vbalayer/special_vbalayerolayer_object.htm`：Material 创建、枚举与存在性。
- `userinterface/elements/backstage.htm`：Pack Project 功能说明。

**已确认：** 外部 MTD 库配置后能显示、左侧 Components 能找到 Solid；官方手册证实 Load、Assign、Update 以及上述接口语义。

**仍需实测：** 跨电脑无外部库情况下的完整材料参数/仿真复现；特定 MTD 格式的外部依赖；是否存在适用于该库的真正“一次批量 Load 所有 MTD”接口。

更多基础术语见 [CST Studio Suite 入门](CST-Studio-Suite.md)。
