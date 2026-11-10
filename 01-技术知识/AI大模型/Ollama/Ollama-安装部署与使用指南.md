---
title: Ollama 安装部署与使用指南
category: 技术知识/AI大模型
tags: [Ollama, LLM, Windows, Linux, GPU, API]
created: 2026-11-10
updated: 2026-11-10
status: maintained
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

## 4. GPU 检查与性能

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

## 5. Windows 模型存储目录迁移

默认位置通常为 `%USERPROFILE%\.ollama\models`。可设置：

```powershell
[Environment]::SetEnvironmentVariable("OLLAMA_MODELS", "D:\Ollama\models", "User")
```

迁移前退出 Ollama，完整备份原目录并复制数据；重启 Ollama 后确认模型可列出、可运行，再考虑删除旧目录。不要直接删除不确定是否被其他模型引用的 blobs。不同版本目录结构可能变化。

## 6. 局域网服务

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

## 7. API 调用

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

## 8. 常见问题

| 现象 | 排查重点 |
| --- | --- |
| 下载缓慢 | 网络、代理环境变量、模型大小；内网可从可信设备迁移完整模型 |
| CPU 推理而非 GPU | 驱动、显存、兼容性；结合 `nvidia-smi`、`ollama ps` 和日志 |
| 浏览器 403/CORS | `OLLAMA_ORIGINS` 是否与前端页面 Origin 一致 |
| 输出截断 | 客户端 `num_predict`、上下文限制、超时、流式响应处理 |
| 复制后模型重复 | 对照 `ollama list`、`ollama show`，先备份再清理标签 |
| Agent 工具调用失败 | 模型是否支持工具调用、JSON Schema、超时和重试逻辑 |

## 9. 与 Local Coding Agent 集成

```text
用户 → Local Coding Agent → Ollama HTTP API → 本地模型 → NVIDIA GPU
             ├─ 文件与 Git
             ├─ 命令行和桌面操作
             └─ 自动验证、失败恢复
```

建议 Agent 层实现工具参数校验、循环检测、失败重试上限、任务状态保存、超时管理和自动化验收。模型的工具调用能力需真实测试，不能仅依据参数规模判断。

## 10. 多机部署建议

多台 Windows + Tesla V100 主机各自运行 Ollama，由上层模型路由服务进行可用性检测、任务分配、超时处理和故障切换。**多台 Ollama 服务的负载均衡不等于跨机器分布式推理。**

## 11. 待补充专题

- Windows 离线安装与模型迁移
- Ollama 代理配置
- CORS 403 完整故障复盘
- Tesla V100 实测性能表
- 多 GPU 与多节点模型路由
- Local Coding Agent 模型兼容性测试

## 12. 更新记录

| 日期 | 版本 | 内容 |
| --- | --- | --- |
| 2026-11-10 | v1.0 | 初始知识库主指南 |
