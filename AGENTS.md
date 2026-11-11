# AI Agent 知识库协作规范

适用于 Codex、Local Coding Agent 及其他在本仓库工作的 AI Agent。

## 操作边界

- 本仓库只存储知识、模板、配置和索引；不要在此添加 Python CLI 源码、测试、构建文件或虚拟环境。
- 开始修改前先读取 README.md、kb.toml，并检索现有知识；不得凭空声称已经检索或验证。
- 对不确定分类的内容先放入 00-inbox，不自动删除原始笔记。
- 不擅自重写、合并或删除现有知识；需要重大改动时先说明影响。
- 新知识遵循 templates/ 中对应模板；区分事实、推测和验证结果。
- 只有实际执行并确认成功的方案才能标记 verified，未经验证使用 draft。
- 不重复复制同一知识；跨分类引用使用相对 Markdown 链接。
- AI 技术知识统一使用 `01-技术知识/AI与大模型/`，禁止新建 `AI大模型/` 等同义分类。
- 解决故障的验证记录归入 `02-问题解决/`；可执行 Skills 归入 `06-AI-Skills/`，避免与 AI 技术原理混放。
- 项目档案每项目一目录，优先使用 overview.md、architecture.md、decisions.md、status.md。
- 归档旧知识时保留原始 ID、来源、归档原因、日期和替代链接（如有）。

## 信息安全

- 当前仓库公开；严禁写入公司内部资料、PCB 原始数据、客户数据、私人信息、凭据、密钥或访问令牌。
- 不得以设置 sensitivity 标签替代公开前审查。
- 遇到疑似敏感信息停止提交并提示人工检查，不在日志中重复输出秘密。

## Git 与校验

- 修改前检查 Git 状态，避免覆盖其他 Agent 未提交的工作。
- 推荐使用独立分支提交，禁止 force push；不得静默解决冲突。
- 更新知识后同步 INDEX.md；若 CLI 未安装，可手工维护或注明待生成。
- 使用可用的校验工具检查链接、文档 ID 和元数据；未运行的测试不得声称通过。
- 不为通过校验而删除失败用例、降低检查标准或虚构结果。

## Markdown 文档统一规范（2026-11-11）

- 正式知识文档（`01-技术知识/` 至 `07-想法与学习/`）必须以 YAML frontmatter 开头；`README.md`、`INDEX.md`、`AGENTS.md` 和 `templates/` 采用各自用途的格式，不强制套用知识文章模板。
- frontmatter 字段按顺序为：`id`、`title`、`category`、`tags`、`status`、`created`、`updated`、`platform`、`source_project`、`source_url`、`sensitivity`。日期使用引号包裹的 `YYYY-MM-DD`；未知可选字段使用空字符串，不编造。
- `id` 全仓唯一且创建后保持稳定；`title` 必须与正文唯一一级标题一致；`category` 使用模板枚举值（technical、troubleshooting、code-snippet、project、procedure、ai-skill、learning），中文业务分类通过目录体现，不混用。
- `status` 采用 `draft`（草稿）、`maintained`（持续维护）、`verified`（真实验证）或 `archived`（归档）；不得因编辑排版就把未验证内容升级为 verified。
- 正文使用一个 `#` 一级标题，`##` 为主章节，`###` 为子章节；同一文章内主章节编号风格保持一致。短篇速查可精简章节，不要求强行填充“原理”等无意义内容。
- 技术概念、使用说明放 `01-技术知识/`；实际故障的复现、修复和验证放 `02-问题解决/`；同一知识只保留一份主文档，其他位置使用相对链接。
- 修改知识文章时保留既有 ID、创建日期、已验证事实和技术内容；仅实际修改时更新 `updated`。同步维护 `INDEX.md`，检查相对链接与重复 ID。
- 创建文档优先复制 `templates/` 对应模板，填充真实元数据；禁止保留模板的空 ID 或占位标题后直接提交。

## 时间导航维护

- 知识库首页 `README.md` 提供 [最近更新](RECENT.md) 与分类索引入口；`INDEX.md` 保留分类导航。
- 新增或实质更新知识文章时，同步维护文章 frontmatter 的 `updated`（`YYYY-MM-DD`）以及 `RECENT.md`，保证导航收录全部正式知识文章，按 `updated` 倒序排列。
- `RECENT.md` 的日期代表文章元数据中的知识维护日期，**不是 GitHub 最后提交时间**；同一天按标题排序，不伪造更精细的时间。
- 不因修改 README、INDEX、RECENT 或单纯格式整理而虚构知识正文更新日期；如需逐次修改历史，引导使用 GitHub History。
- 在自动生成工具落地前，`RECENT.md` 为人工维护的静态页面；不得声称会自动刷新。
