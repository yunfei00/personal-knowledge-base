---
id: tech-ai-ollama-001
title: Ollama 安装部署与使用指南
category: technical
tags: [Ollama, LLM, Windows, Linux, GPU, API]
status: maintained
created: "2026-11-10"
updated: "2026-11-11"
platform: "Windows 10/11 / Ubuntu / NVIDIA GPU"
source_project: ""
source_url: "https://docs.ollama.com"
sensitivity: public
---

# Ollama 安装部署与使用指南

> 适用：Windows 10/11、Ubuntu、NVIDIA Tesla V100 等本地模型部署环境。本文为主指南；具体故障建议另建独立问题记录。

## 1. 简介

[Ollama](https://ollama.com) 是本地运行和管理大语言模型的工具，本身不是大语言模型。支持模型下载、运行、API 服务以及部分模型的视觉和工具调用能力（以具体模型为准）。

- 官方文档：https://docs.ollama.com
- 模型库：https://ollama.com/library
- GitHub：https://github.com/ollama/ollama

## 2. 安装与验证

### Windows

从 https://ollama.com/download 下载 Windows 安装包，安装后在 PowerShell 中运行：

```powershell
ollama --version
ollama list
ollama ps
```

### Ubuntu

先审查安装脚本，再运行官方安装命令：

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama --version
systemctl status ollama
sudo systemctl enable --now ollama
```

## 3. 模型管理

```powershell
ollama pull gpt-oss:20b
ollama pull nomic-embed-text
ollama list
ollama show gpt-oss:20b
ollama run gpt-oss:20b
ollama ps
ollama stop gpt-oss:20b
# 确认不再使用时才删除：
# ollama rm gpt-oss:20b
```

在交互界面输入 `/bye` 退出。模型名称、版本及能力应以当前模型库为准。

### 自定义 Modelfile

```text
FROM gpt-oss:20b
PARAMETER temperature 0.2
PARAMETER num_ctx 8192
```

保存为 `Modelfile`，然后运行 `ollama create gpt-oss-custom -f Modelfile`。较大的上下文通常增加显存占用。

## 4. 模型加载、显存驻留与耗电

### 4.1 下载、加载和推理不是同一件事

| 阶段 | 发生什么 | GPU 显存状态 |
| --- | --- | --- |
| `ollama pull` | 下载模型文件到硬盘 | 通常不会因此把模型权重常驻显存 |
| 首次 `ollama run` 或 API 请求 | 服务端按需加载模型到 GPU 显存和/或系统内存 | 显存占用增加；首次调用有加载延迟 |
| 模型推理 | GPU/CPU 执行计算并生成回答 | 计算利用率、功耗通常上升 |
| 请求结束后空闲 | 模型可继续驻留，等待后续请求复用 | 可能仍占用显存，但不等于 GPU 满负载 |
| 驻留期满或手动停止 | 卸载模型，释放相关资源 | 硬盘模型文件不会因此删除 |

本机调用和局域网调用遵循同一加载机制：客户端通过 HTTP 请求 Ollama 服务端，模型实际运行在**服务端** GPU 上。多个客户端调用同一服务端，不需要各自在本机下载模型。若模型已经加载且未被卸载，后续请求通常可以省去加载时间；但高并发仍受显存、上下文和服务端调度限制。

### 4.2 驻留时间在哪里设置？

**全局默认值：** 环境变量 `OLLAMA_KEEP_ALIVE`。常见默认驻留时间是空闲约 5 分钟（以实际版本及配置为准）。

| 值 | 含义 |
| --- | --- |
| `0` | 请求完成后尽快卸载 |
| `5m` | 空闲保留约 5 分钟 |
| `30m` | 空闲保留约 30 分钟 |
| `1h` | 空闲保留约 1 小时 |
| `-1` | 持续驻留，直至主动卸载、服务退出或资源调度要求卸载 |

Windows 桌面版示例（用户级环境变量）：

```powershell
# 为后续启动的 Ollama 进程设置默认驻留 30 分钟
[Environment]::SetEnvironmentVariable("OLLAMA_KEEP_ALIVE", "30m", "User")

# 查看保存的用户级设置
[Environment]::GetEnvironmentVariable("OLLAMA_KEEP_ALIVE", "User")

# 查看当前已加载模型及预计卸载时间
ollama ps
```

**设置后要完全退出并重新启动 Ollama**，让新进程读取环境变量。若作为 Windows 服务或 Linux systemd 服务运行，应在对应服务环境配置并重启服务，而不是只修改当前用户的变量。

**单次 API 请求覆盖默认值：**

```json
{
  "model": "gpt-oss:20b",
  "prompt": "你好",
  "stream": false,
  "keep_alive": "30m"
}
```

`POST /api/generate` 或 `POST /api/chat` 可传入 `keep_alive`；请求级设置优先于全局默认值。可以用不带 `prompt` 的 `/api/generate` 请求预加载模型，并设置驻留时间。通过 `ollama ps` 查看 `UNTIL` 列确认预计卸载时间。驻留并非显存锁定保证：服务退出或资源调度仍可能导致模型卸载。

### 4.3 常驻模型有什么代价？

| 对比 | 长时间驻留 | 按需加载 |
| --- | --- | --- |
| 首次响应 | 通常更快 | 需要等待模型加载 |
| 空闲显存 | 持续占用 | 卸载后释放 |
| 空闲 GPU 功耗 | 可能更高，需实测 | 通常更低 |
| 推理性能 | 相同条件下不会因驻留而自动更快 | 加载完成后通常相近 |
| 运行其他模型 / CUDA 任务 | 可能出现显存竞争 | 显存更灵活 |

**显存占用高不等于 GPU 利用率高，更不等于持续满功率耗电。** 常驻可能增加 GPU 待机功耗、温度及风扇负担，但幅度受硬件、驱动、电源状态影响；不能直接把模型大小换算为电费。模型文件保存在硬盘，与是否常驻无关。

计算额外电费的示例公式：

```text
每月额外耗电（kWh）= 额外空闲功耗（W）× 每天驻留小时数 × 30 ÷ 1000
每月额外电费 = 每月额外耗电（kWh）× 每度电价
```

例如**假设**额外空闲功耗 20W、每天 24 小时、30 天、电价 0.6 元/kWh，则额外耗电 14.4 kWh、电费约 8.64 元；这不是 Tesla V100 的实测数据。

### 4.4 如何验证驻留与实际功耗？

在 **Ollama 服务端** 的 Windows PowerShell 中操作，替换成真实存在的模型名称：

```powershell
$model = "gpt-oss:20b"

# 卸载后测量空闲基线
ollama stop $model
Start-Sleep -Seconds 30
nvidia-smi --query-gpu=name,power.draw,memory.used,utilization.gpu,temperature.gpu --format=csv

# 通过本机 API 预加载并设置长时间驻留（无 prompt）
$body = @{ model = $model; keep_alive = -1 } | ConvertTo-Json
Invoke-RestMethod -Uri "http://127.0.0.1:11434/api/generate" -Method Post -ContentType "application/json" -Body $body

# 等待推理活动结束后再观察
Start-Sleep -Seconds 30
ollama ps
nvidia-smi --query-gpu=name,power.draw,memory.used,utilization.gpu,temperature.gpu --format=csv

# 测试结束后释放显存
ollama stop $model
```

建议两种状态各连续测几分钟、取平均值，并确保没有其他 GPU 任务干扰。`nvidia-smi` 主要反映 GPU 板卡功耗，不是整机墙上插座功耗；要估算整机电费，使用插座功率计更准确。

### 4.5 实用建议

- **偶尔调用、多模型切换：** 保持默认 5 分钟或使用 30 分钟驻留。
- **Local Coding Agent 连续开发：** 可先试 30 分钟，减少反复加载。
- **专用局域网推理服务器：** 高频使用同一模型时可以考虑 `-1`，但要监控显存与空闲功耗。
- **Tesla V100 32GB：** 优先实测 `ollama ps` 的加载情况、`nvidia-smi` 的显存和功耗；不要仅凭模型文件大小判断是否能与其他模型同时驻留。

## 5. GPU 检查与性能

```powershell
nvidia-smi
ollama ps
```

`ollama ps` 的 PROCESSOR 字段用于判断 CPU/GPU 加载情况。Tesla V100 32GB 建议从 8192 上下文开始测试，并记录显存占用、首字延迟、输出速度和任务正确率。双 V100 可尝试多卡分担模型，但不等于一张统一的 64GB GPU。

Ollama API 返回的 `eval_count` 与 `eval_duration`（纳秒）可计算：

```text
tokens/s = eval_count * 1_000_000_000 / eval_duration
```

对编程 Agent，工具调用成功率和任务完成率比单独的 tokens/s 更重要。

## 6. Windows 模型存储目录迁移

默认位置通常为 `%USERPROFILE%\.ollama\models`。可设置：

```powershell
[Environment]::SetEnvironmentVariable("OLLAMA_MODELS", "D:\Ollama\models", "User")
```

迁移前退出 Ollama，完整备份原目录并复制数据；重启 Ollama 后确认模型可列出、可运行，再考虑删除旧目录。不要直接删除不确定是否被其他模型引用的 blobs。不同版本目录结构可能变化。

## 7. 局域网服务

默认 API 地址通常为 `http://127.0.0.1:11434`。Windows 设置中可开启 **Expose Ollama to the network**，也可设置：

```powershell
[Environment]::SetEnvironmentVariable("OLLAMA_HOST", "0.0.0.0:11434", "User")
```

退出并重新启动 Ollama 后，用 `http://服务器IP:11434/api/tags` 验证。防火墙只允许可信设备访问；Ollama 原生 API 通常不提供内置身份认证，**不要直接暴露到公网**。

### 浏览器 CORS / 403

跨域来源与监听地址是不同配置。按实际前端 Origin 设置白名单，例如：

```powershell
[Environment]::SetEnvironmentVariable("OLLAMA_ORIGINS", "http://192.168.1.100:3000", "User")
```

重新启动后测试。不要为省事而对不可信来源全面开放。

## 8. API 调用

常见接口：

| 接口 | 用途 |
| --- | --- |
| `GET /api/tags` | 查看模型 |
| `POST /api/generate` | 文本生成 |
| `POST /api/chat` | 对话 |
| `POST /api/embed` | 向量嵌入 |
| `/v1` | 部分 OpenAI 兼容 API |

Python 示例（先安装 `requests`）：

```python
import requests

r = requests.post(
    "http://127.0.0.1:11434/api/chat",
    json={
        "model": "gpt-oss:20b",
        "messages": [{"role": "user", "content": "介绍一下 Ollama"}],
        "stream": False,
    },
    timeout=180,
)
r.raise_for_status()
print(r.json()["message"]["content"])
```

## 9. 常见问题

| 现象 | 排查重点 |
| --- | --- |
| 下载缓慢 | 网络、代理环境变量、模型大小；内网可从可信设备迁移完整模型 |
| CPU 推理而非 GPU | 驱动、显存、兼容性；结合 `nvidia-smi`、`ollama ps` 和日志 |
| 浏览器 403/CORS | `OLLAMA_ORIGINS` 是否与前端页面 Origin 一致 |
| 输出截断 | 客户端 `num_predict`、上下文限制、超时、流式响应处理 |
| 复制后模型重复 | 对照 `ollama list`、`ollama show`，先备份再清理标签 |
| Agent 工具调用失败 | 模型是否支持工具调用、JSON Schema、超时和重试逻辑 |

## 10. 与 Local Coding Agent 集成

```text
用户 → Local Coding Agent → Ollama HTTP API → 本地模型 → NVIDIA GPU
             ├─ 文件与 Git
             ├─ 命令行和桌面操作
             └─ 自动验证、失败恢复
```

建议 Agent 层实现工具参数校验、循环检测、失败重试上限、任务状态保存、超时管理和自动化验收。模型的工具调用能力需真实测试，不能仅依据参数规模判断。

## 11. 多机部署建议

多台 Windows + Tesla V100 主机各自运行 Ollama，由上层模型路由服务进行可用性检测、任务分配、超时处理和故障切换。**多台 Ollama 服务的负载均衡不等于跨机器分布式推理。**

## 12. 待补充专题

- Windows 离线安装与模型迁移
- Ollama 代理配置
- CORS 403 完整故障复盘
- Tesla V100 实测性能表
- 多 GPU 与多节点模型路由
- Local Coding Agent 模型兼容性测试

## 13. 更新记录

| 日期 | 版本 | 内容 |
| --- | --- | --- |
| 2026-11-10 | v1.0 | 初始知识库主指南 |
| 2026-11-11 | v1.1 | 新增模型按需加载、局域网调用、驻留时间设置、GPU 功耗与实测方法；按使用流程调整章节 |
