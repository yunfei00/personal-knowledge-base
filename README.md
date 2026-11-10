# Personal Knowledge Base

个人知识库，使用 Markdown + Git 长期积累可复用的技术知识、故障解决方案、代码片段、项目档案、标准流程、AI Skills 和学习记录。

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

## 使用原则

1. 先检索，再新增；一份知识一个主要归档位置，其他位置用相对链接引用。
2. 问题解决记录真实的验证过程，未验证的方案不能标记为已验证。
3. 每周整理 Inbox，不因到期而自动删除。
4. 公开仓库只允许提交可公开内容；公司内部数据、账号密钥、客户信息等禁止提交。
5. 原仓库 `yunfei00/document` 暂不迁移，迁移前单独审计。

## 工具

知识数据与 Python 工具独立。未来使用 `personal-kb-cli` 提供的 `kb check` 和 `kb index` 校验、生成索引。工具未发布前无需安装。

详见 [AGENTS.md](AGENTS.md)、[kb.toml](kb.toml) 和 [INDEX.md](INDEX.md)。
