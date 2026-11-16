---
id: tech-database-postgresql-001
title: PostgreSQL 入门：数据库作用、Docker 部署与数据持久化
category: technical
tags: [PostgreSQL, Postgres, 数据库, SQL, Docker, Docker Compose, Django, Redis, Celery, 数据备份]
status: maintained
created: "2026-11-16"
updated: "2026-11-16"
platform: "Windows / Linux / Docker"
source_project: ""
source_url: "https://www.postgresql.org/docs/"
sensitivity: public
---

# PostgreSQL 入门：数据库作用、Docker 部署与数据持久化

## 1. PostgreSQL 是什么？

**PostgreSQL**（常简称 **Postgres**）是一款开源的对象关系型数据库管理系统（DBMS）。它负责将应用数据长期保存下来，并提供查询、更新、关联、事务处理、权限控制和备份恢复等能力。

区分三个概念：

- **PostgreSQL**：数据库服务软件，负责管理数据。
- **数据库（database）**：PostgreSQL 实例中存放业务表等对象的逻辑空间。
- **Postgres Docker 镜像**：打包好的 PostgreSQL 运行环境；容器运行镜像后才成为数据库服务。

## 2. 主要用途

| 场景 | PostgreSQL 保存或处理什么 |
| --- | --- |
| 用户系统 | 用户资料、密码哈希、角色权限 |
| 数据采集 | 来源、采集记录、原始数据、去重标识 |
| 任务系统 | 任务配置、执行状态、失败原因、历史记录 |
| AI 分析平台 | 结构化分析结果、标签、报表数据 |
| 业务统计 | 按日期、状态、类型分组、过滤、聚合 |
| 数据一致性 | 使用事务保证一组操作同时成功或回滚 |

SQL 示例（假设业务系统中已存在 `tasks` 表）：

```sql
SELECT * FROM tasks ORDER BY created_at DESC LIMIT 10;
SELECT status, COUNT(*) FROM tasks GROUP BY status;
```

PostgreSQL 还支持索引、外键、JSON/JSONB、复杂查询等能力。

## 3. PostgreSQL 与 Redis、Django、Celery 的关系

| 组件 | 核心职责 |
| --- | --- |
| Django | Web/API 业务逻辑，通过 ORM 或 SQL 访问数据库 |
| PostgreSQL | 持久化保存业务数据和查询结果 |
| Redis | 高速键值存储，常用于缓存、Celery 消息代理 |
| Celery Worker | 异步执行采集、分析等后台任务 |
| Docker Compose | 编排和启动多个容器服务 |

典型流程：浏览器请求 Django → Django 创建任务记录到 PostgreSQL → 将异步任务交给 Celery（例如通过 Redis）→ Worker 执行采集或分析 → 结果写回 PostgreSQL → Django 查询并展示。

注意：Redis 也支持持久化，PostgreSQL 也有缓存机制；上述区分是典型职责分工，不是绝对限制。

## 4. 使用 Docker Compose 部署

以下示例适合本地学习或内部测试。生产环境需要单独考虑密码管理、备份、网络访问和版本升级。

```yaml
services:
  postgres:
    image: postgres:17
    restart: unless-stopped
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?请在 .env 中设置 POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 10s
      timeout: 5s
      retries: 5
    # 不配置 ports 时，数据库不直接暴露到宿主机端口

volumes:
  postgres_data:
```

在与 `compose.yaml` 同目录的 `.env` 中设置 `POSTGRES_PASSWORD`，不要把真实密码提交到公开仓库。数据库容器默认监听 5432 端口，同一 Compose 网络中的其他服务可使用主机名 `postgres` 连接。

**版本注意：** 上例使用 `postgres:17`，其数据目录示例为 `/var/lib/postgresql/data`。不同 PostgreSQL 主版本的官方镜像数据目录约定可能变化，升级镜像前必须核对官方文档，不要直接跨主版本复用旧数据文件。

常用命令（在 Compose 项目目录运行）：

```powershell
docker compose up -d postgres
docker compose ps
docker compose logs --tail=100 postgres
docker compose exec postgres psql -U appuser -d appdb
```

进入 `psql` 后，可使用 `\dt` 查看表，`\l` 查看数据库，`\q` 退出。

## 5. Docker 数据持久化与备份

`postgres_data` 是 Docker 命名卷，容器重建后通常仍可复用其中的数据。

- `docker compose down`：通常删除容器和网络，**不删除命名卷**。
- `docker compose down -v`：会删除 Compose 管理的命名卷，**可能永久丢失数据库数据**。
- 仅有数据卷不等于已经做了备份。需要定期生成备份文件并测试恢复。
- 数据库运行时，不建议直接复制活动数据目录作为唯一备份手段。

逻辑备份示例（Windows PowerShell；先在容器内生成二进制备份，再复制到宿主机，避免旧版 PowerShell 重定向损坏二进制文件）：

```powershell
New-Item -ItemType Directory -Force -Path .\backups | Out-Null
docker compose exec -T postgres pg_dump -U appuser -d appdb -Fc -f /tmp/appdb.dump
$containerId = docker compose ps -q postgres
docker cp "${containerId}:/tmp/appdb.dump" .\backups\appdb.dump
docker compose exec -T postgres rm -f /tmp/appdb.dump
```

执行时应检查各命令的退出状态；恢复前应确认备份文件有效，并先在测试数据库演练。上面的示例用于理解流程，正式环境应配合备份调度、保留策略和异地存储。

## 6. 常见问题

**Q：Docker 中出现 `postgres` 容器，是否表示系统使用 PostgreSQL？**

通常是。可查看 `compose.yaml` 中的 `image`、`environment`、`volumes` 和应用的数据库连接配置，确认其用途。

**Q：为什么删除容器后数据还在？**

因为数据通常保存在独立的命名卷或宿主机挂载目录中，而不是只保存在容器可写层。

**Q：能否用 Redis 替代 PostgreSQL？**

不能简单一对一替代。复杂关联查询、事务性业务记录通常更适合 PostgreSQL；缓存和消息代理等通常更适合 Redis。

**Q：`postgres` 和 `PostgreSQL` 是两个数据库吗？**

不是，前者是后者的常用简称，也常用作 Docker 镜像名、服务名或默认管理员角色名。

## 7. 安全和运维注意事项

1. 不要将数据库 5432 端口无必要地暴露到公网。
2. 使用独立账号、强密码和最小权限；不要把真实凭据写入 Git。
3. 重要业务配置自动备份、恢复演练和监控。
4. PostgreSQL 主版本升级前先核对数据格式兼容性和迁移流程。
5. 数据库报错时优先查看容器日志、连接参数、磁盘空间和数据卷状态，不要直接删除数据卷。

## 8. 参考资料

- [PostgreSQL 官方文档](https://www.postgresql.org/docs/)
- [Postgres Docker 官方镜像说明](https://hub.docker.com/_/postgres)
- [Docker Compose volumes 文档](https://docs.docker.com/reference/compose-file/volumes/)
