---
id: tech-cad-creo-001
title: Creo Parametric 入门：三维 CAD、零件、装配与特征
category: technical
tags: [Creo, Creo Parametric, CAD, 三维建模, Part, Assembly, Feature, SAT]
status: draft
created: "2026-11-11"
updated: "2026-11-11"
platform: "Windows / Creo Parametric（版本待确认）"
source_project: ""
source_url: "https://www.ptc.com/en/products/creo"
sensitivity: public
---

# Creo Parametric 入门：三维 CAD、零件、装配与特征

> 本文是持续维护的 Creo 基础知识笔记。先解释常见概念及与 CST 的关系，后续遇到实际问题时继续补充。这里主要介绍 **Creo Parametric**，不把 Creo 产品家族的全部功能混为一谈。

## 1. Creo 是什么？

**Creo** 是 PTC 推出的计算机辅助设计（CAD）软件产品家族。其中 **Creo Parametric** 主要用于机械零件的参数化三维建模、装配设计和工程图制作，常见于机械结构、电子产品结构件、模具和工业产品设计。

简单理解：

- **Creo：** 设计产品的几何结构，表达零件尺寸和装配关系。
- **CST Studio Suite：** 利用几何结构、材料参数及仿真设置计算电磁问题。

两者可以配合使用，但各自的工程数据、建模对象和材料定义并不完全相同。

## 2. Creo 常见术语

| 英文 | 中文 | 简单解释 |
| --- | --- | --- |
| Part | 零件 | 单个零件的设计模型 |
| Assembly | 装配体 | 将多个零件或子装配组织在一起 |
| Feature | 特征 | 拉伸、旋转、孔、倒角等建模操作 |
| Body | 几何实体体 | 一个零件模型中可存在的实体几何体 |
| Sketch | 草绘 | 创建几何轮廓的二维草图 |
| Datum | 基准 | 基准平面、轴、点、坐标系等 |
| Constraint | 约束 | 控制草绘或装配关系的条件 |
| Material | 材料 | 可关联材料属性的模型信息 |
| Drawing | 工程图 | 用于标注尺寸、公差和制造信息的图纸 |
| Regenerate | 再生 | 根据参数及特征依赖关系更新模型 |

## 3. Part（零件）是什么？

**Part** 是 Creo Parametric 中常见的零件设计单元，通常对应一个 `.prt` 类型的零件文件。

例如：手机金属中框、塑料支架、螺钉等，都可能分别设计为 Part。

**注意：** 一个 Part 不一定只包含一个 Body，也不能直接把“一个 Part”理解成“一个物料编号”。实际物料关系取决于设计和 BOM 管理方式。

## 4. Assembly（装配体）是什么？

**Assembly** 是把多个 Part 或子装配组合起来的模型，常见文件扩展名为 `.asm`。

```text
手机结构装配（Assembly）
├── 中框（Part）
├── 后盖（Part）
├── 支架（Part）
└── 屏蔽罩组件（Subassembly）
    ├── 屏蔽罩 A（Part）
    └── 屏蔽罩 B（Part）
```

装配体包含零件的层级与定位关系，但导出为中性几何格式后，部分装配语义、参数特征或属性可能无法完整保留。

## 5. Feature（特征）是什么？

**Feature** 是参数化建模的基本构造步骤。一个零件通常由多个特征按依赖关系逐步构建：

```text
草绘（Sketch）
   ↓
拉伸（Extrude）
   ↓
打孔（Hole）
   ↓
倒角（Chamfer）
   ↓
圆角（Round）
   ↓
最终零件几何形状
```

Creo 的模型树可用于查看和管理这些特征。修改某个尺寸后，相关特征通常需要重新计算或再生。

## 6. Body 和 Part 有什么区别？

- **Part：** 零件文档与设计单元。
- **Body：** Part 内的实体几何体。
- **Feature：** 生成或修改几何的建模步骤。

因此，Part、Body、Feature 不是同一级概念。导出三维结构到其他软件后，通常更容易保留的是几何体，而不是完整的原始参数化特征历史。

## 7. Creo 中的 Material（材料）

Creo 可为模型关联材料信息，但**是否有材料、材料属性是否完整、导出时是否保留**，需要根据具体工程和格式检查。

特别是当 Creo 工程只有几何结构、没有可靠材料属性时：

1. 不应从文件名或零件名称直接推断材料真值。
2. 不应认为导入 CST 后材料会自动正确。
3. 可以利用尺寸、厚度、形状、空间位置和装配关系辅助识别结构功能。
4. 最终材料赋值应依据已经确认的工程知识、材料定义和验证规则。

**几何识别与材料赋值是两个不同任务。**

## 8. 常见文件格式

| 格式 | 常见用途 | 注意事项 |
| --- | --- | --- |
| `.prt` | Creo 零件文件 | 通常保留原生零件设计信息 |
| `.asm` | Creo 装配文件 | 依赖关联零件文件及装配关系 |
| `.drw` | Creo 工程图 | 用于图纸表达 |
| STEP（`.stp` / `.step`） | CAD 几何交换 | 具体保留内容取决于导出选项及格式版本 |
| IGES（`.igs` / `.iges`） | CAD 几何交换 | 常用于曲面、几何数据交换 |
| STL（`.stl`） | 三角网格模型 | 不等同于可编辑的精确参数化实体 |
| SAT（`.sat`） | ACIS 几何交换 | 需要确认 Creo 版本及所用转换链路是否支持 |

**不要把 SAT 当作所有 Creo 版本必然直接支持的原生导出格式。** 具体能力应以当前安装版本、模块及实际导入导出验证为准。

## 9. Creo 与 CST 如何配合？

一种典型工作链路：

```text
Creo Parametric
  └── 设计零件与装配体
        ↓
    导出可交换的几何格式
        ↓
CST Studio Suite
  ├── 导入三维几何
  ├── 检查 Solid / Component
  ├── 确认或赋予 Material
  ├── 设置求解器、激励、边界等
  └── 保存工程并开展仿真
```

两套软件中的对象名称和层级不一定一一对应：

| Creo 概念 | CST 中可能的对应概念 | 是否严格一一对应 |
| --- | --- | --- |
| Part | 一个或多个 Solid | 否 |
| Assembly | Component 层级或导入后的组织结构 | 否 |
| Body | 导入后的几何实体 | 不保证 |
| Material | CST Material | 不保证自动传递 |

**重点：** 对自动化识别工具而言，应该验证导入后的真实几何对象、尺寸、位置和关联关系，而不是假设 Creo 文件中的每个零件都必然成为 CST 的一个 Solid。

相关文档：[CST Studio Suite 入门](../../仿真与CAE/CST/CST-Studio-Suite.md)。

## 10. Creo 与其他软件的区别

| 软件 | 主要定位 |
| --- | --- |
| Creo Parametric | 参数化机械 CAD、零件与装配设计 |
| SolidWorks | 三维机械 CAD 设计 |
| Autodesk Inventor | 机械 CAD 设计 |
| CST Studio Suite | 电磁场仿真 |
| Ansys HFSS | 高频电磁场仿真 |

CAD 软件用于建立设计模型，CAE 软件用于分析和验证物理行为；实际产品和模块可能存在功能交叉。

## 11. 后续待补充

- Creo 模型树怎么看。
- Part、Body、Feature 的实际界面操作。
- 如何查看几何尺寸、体积和装配位置。
- 如何检查模型中是否存在材料属性。
- STEP / SAT 等几何交换格式的实际导出路径。
- Creo 结构导入 CST 后对象数量与层级变化。
- 如何在不依赖文件名的前提下识别结构几何特征。

遇到问题后优先扩充本文；较长的故障解决过程另存到 `02-问题解决/`，并在这里建立链接。

## 参考资料

- [PTC Creo 官方产品介绍](https://www.ptc.com/en/products/creo)
- [PTC 官方支持与文档入口](https://support.ptc.com/)
