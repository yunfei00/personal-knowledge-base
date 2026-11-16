---
id: tech-system-windows-screenshot-001
title: Windows 截屏与截图快捷键速查
category: technical
tags: [Windows, 截图, 截屏, Snipping Tool, 快捷键, 剪贴板]
status: maintained
created: "2026-11-16"
updated: "2026-11-16"
platform: "Windows 10 / Windows 11"
source_project: ""
source_url: ""
sensitivity: public
---

# Windows 截屏与截图快捷键速查

> **一句话：** 日常截图优先记住 **`Win + Shift + S`**；它可以快速截取屏幕的一部分，并直接复制到剪贴板。

## 1. 最常用：Win + Shift + S

按下：

```text
Win + Shift + S
```

屏幕会变暗，并打开 Windows 自带的截图工具栏。通常可以选择：

- **矩形截图**：拖动鼠标框选一个矩形区域。
- **自由形状截图**：手动画出需要截取的范围。
- **窗口截图**：直接截取某一个应用窗口。
- **全屏截图**：截取整个屏幕。

截图完成后，图片会进入**剪贴板**，可以直接在微信、QQ、Word、画图、浏览器或 ChatGPT 中按：

```text
Ctrl + V
```

进行粘贴。

这是日常最推荐的截图方式。

## 2. 常用截图快捷键

| 快捷键 | 作用 | 是否自动保存 |
| --- | --- | --- |
| `Win + Shift + S` | 打开截图工具，可截区域、窗口或全屏 | 至少复制到剪贴板；是否自动保存取决于截图工具设置 |
| `Win + PrtScn` | 截取整个屏幕 | **会自动保存** |
| `PrtScn` | 截取整个屏幕，或根据系统设置打开截图工具 | 通常进入剪贴板 |
| `Alt + PrtScn` | 只截当前活动窗口 | 进入剪贴板 |
| `Win + Alt + PrtScn` | 使用 Xbox Game Bar 截取当前游戏或应用 | 自动保存到“捕获”目录 |

> 部分笔记本电脑的 `PrtScn` 与其他功能共用一个按键，需要同时按 `Fn`，例如 `Fn + PrtScn`。

## 3. Win + PrtScn 截图保存在哪里

按：

```text
Win + PrtScn
```

Windows 会直接截取整个屏幕，并自动保存图片。

通常保存目录为：

```text
C:\Users\你的用户名\Pictures\Screenshots
```

中文 Windows 中一般可以从资源管理器进入：

```text
图片 → 屏幕截图
```

这种方式适合需要**连续截图并保留文件**的场景。

## 4. PrtScn 和 Alt + PrtScn

### 4.1 PrtScn

直接按：

```text
PrtScn
```

常见行为有两种：

1. 截取整个屏幕到剪贴板；
2. Windows 11 中，如果启用了“使用 Print Screen 键打开屏幕截图”选项，则会直接打开截图工具。

截完后可以打开画图、Word、微信等应用，按 `Ctrl + V` 粘贴。

### 4.2 Alt + PrtScn

按：

```text
Alt + PrtScn
```

只截取**当前正在使用的窗口**，不会把整个桌面都截进去。

例如当前正在使用 PyCharm，只想截 PyCharm 窗口时，这个组合键很方便。

## 5. 使用 Windows“截图工具”

Windows 10/11 都自带 **截图工具（Snipping Tool）**。

打开方式：

1. 按 `Win` 键；
2. 搜索“截图工具”或“Snipping Tool”；
3. 打开后点击“新建”进行截图。

截图工具适合需要进一步进行以下操作的情况：

- 延迟截图；
- 标注；
- 裁剪；
- 保存为图片文件；
- 复制到剪贴板。

## 6. 截图后直接发给别人

如果只是想截一小块区域发到聊天软件或 ChatGPT，最快流程是：

```text
Win + Shift + S
→ 鼠标框选区域
→ 切换到聊天窗口
→ Ctrl + V
```

不需要先保存成文件。

## 7. 怎么选

日常使用可以只记住下面三个：

| 需求 | 推荐方式 |
| --- | --- |
| 截一小块区域 | **Win + Shift + S** |
| 整个屏幕并自动保存 | **Win + PrtScn** |
| 只截当前窗口 | **Alt + PrtScn** |

其中最值得记住的是：

```text
Win + Shift + S
```

它基本可以覆盖绝大多数日常截图需求。

## 8. 注意事项

- 多显示器环境下，全屏截图可能会把多个屏幕一起截进去；需要单独截某一块时优先使用 `Win + Shift + S`。
- 部分笔记本需要配合 `Fn` 键使用 `PrtScn`。
- 截图进入剪贴板后，如果还没有保存文件，后续复制其他内容可能覆盖剪贴板中的图片。
- Windows 不同版本和设置下，`PrtScn` 的默认行为可能不同；不确定时直接使用 `Win + Shift + S` 最稳妥。
