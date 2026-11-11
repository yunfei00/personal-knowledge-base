# Personal Knowledge Base

个人知识库，使用 Markdown + Git 长期积累可复用的技术知识、故障解决方案、代码片段、项目档案、标准流程、AI Skills 和学习记录。

## 快速导航

- 📅 [最近更新 · 按修改日期从近到远复习](RECENT.md)（可直接点击文章）
- 📚 [知识分类索引](INDEX.md)（按主题查找）
- 🕘 [GitHub 提交历史](https://github.com/yunfei00/personal-knowledge-base/commits/main/)（查看每次真实提交）

## 分类

- `00-inbox/`：临时收集，每周整理，不强制删除未处理内容
- `01-技术知识/`：技术概念、原理、使用方法，按领域分类
- `02-问题解决/`：现象、根因、修复、验证；所有故障统一归档
- `03-代码片段/`：可复用短代码及用法
- `04-项目档案/`：一个项目一个文件夹，建议包含 overview.md、architecture.md、decisions.md、status.md
- `05-标准流程/`：开发、测试、CI/CD、发布、部署等可重复步骤
- `06-AI-Skills/`：agents、skills、prompts、workflows
- `07-想法与学习/`：灵感、计划和实验
- `99-archive/`：过时、废弃但仍有历史价值的知识
- `templates/`：Markdown 文档模板

## AI 分类约定

- AI 相关技术统一归档到 `01-技术知识/AI与大模型/`，不再新建 `AI大模型/` 等同义目录。
- Ollama、模型部署、RAG、AI Agent、多模态和模型评测按具体主题细分；目录不必预先创建空文件夹。
- 已验证的故障解决方案放在 `02-问题解决/`，AI Skills 可执行资产放在 `06-AI-Skills/`，主指南通过相对链接引用，避免复制内容。
- 新增子目录前先检查是否已有同义分类。

## 使用原则

1. 先检索，再新增；一份知识一个主要归档位置，其他位置用相对链接引用。
2. 问题解决记录真实的验证过程，未验证的方案不能标记为已验证。
3. 每周整理 Inbox，不因到期而自动删除。
4. 公开仓库只允许提交可公开内容；公司内部数据、账号密钥、客户信息等禁止提交。
5. 原仓库 `yunfei00/document` 暂不迁移，迁移前单独审计。

## 工具

知识数据与 Python 工具独立。未来使用 `personal-kb-cli` 提供的 `kb check` 和 `kb index` 校验、生成索引。工具未发布前无需安装。

详见 [AGENTS.md](AGENTS.md)、[kb.toml](kb.toml) 和 [INDEX.md](INDEX.md)。
