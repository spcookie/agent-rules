# Agent Rules

为 Codex、Claude、OpenCode、Pi 等编码 Agent 维护的共享工程规范。仓库内的规则是唯一原稿；各项目通过 Git 引用固定版本，避免在不同
Agent 的入口文件里重复维护规则正文。

## 规则索引

规则正文按专题集中在 [`rules/`](rules/) 目录中，本 README 是统一入口。

### 适用范围

- 开始任务前，必须先读取并遵守 [`rules/general.md`](rules/general.md)。
- 根据任务涉及的领域，读取下表中的全部相关规则；跨领域任务需要同时读取多个文件。
- 已有项目优先保持现行结构，在改动相关模块时逐步靠拢这些规范；不要为了符合目录模板而无意义地拆分文件或模块。
- 专题规则可由项目级约定进一步细化。若规则之间或规则与项目现有约定冲突，先说明取舍并与项目所有者确认，不得自行覆盖决策约束。

| 任务领域                        | 必读规则                                                   | 内容                                                      |
|---------------------------------|------------------------------------------------------------|-----------------------------------------------------------|
| 所有任务                        | [`rules/general.md`](rules/general.md)                     | 基本原则、技术选型与变更边界                              |
| 前端                            | [`rules/frontend.md`](rules/frontend.md)                   | 前端技术栈、目录结构、状态与展示职责                      |
| 后端                            | [`rules/backend.md`](rules/backend.md)                     | 后端技术栈、六边形 / 洋葱架构与事务边界                   |
| 数据库、缓存、消息或后台任务    | [`rules/middleware.md`](rules/middleware.md)               | PostgreSQL、Caffeine、Redis、RabbitMQ、JobRunr 的使用边界 |
| HTTP API、gRPC 或消息格式       | [`rules/contracts.md`](rules/contracts.md)                 | OpenAPI、Proto、消息契约、兼容性与契约校验                |
| API、错误处理、国际化或可观测性 | [`rules/api-observability.md`](rules/api-observability.md) | DTO、错误模型、OpenTelemetry、日志与多语言约定            |
| 代码变更、CI 或交付             | [`rules/quality.md`](rules/quality.md)                     | lint、格式化、质量门控、测试与交付要求                    |

### 读取示例

- 前端页面改动：`general.md`、`frontend.md`、`quality.md`。
- 后端 API 改动：`general.md`、`backend.md`、`contracts.md`、`api-observability.md`、`quality.md`；涉及数据库或消息时再读取`middleware.md`。
- 仅修改文档：至少读取 `general.md`，再按文档覆盖的主题读取对应规则。

## 在项目中引用

推荐将本仓库作为项目的 Git submodule 引入，使每个项目固定规则版本，规则更新通过项目自身的 PR 审核：

```bash
git submodule add -b master https://github.com/spcookie/agent-rules.git .agent-rules
git commit -m "Add shared agent rules"
```

在项目根目录的 `AGENTS.md` 中加入以下入口，供 Codex、OpenCode、Pi 使用：

```md
# Project instructions

开始任务前，读取并遵守 `.agent-rules/README.md`，再按其中的索引读取
`.agent-rules/rules/` 下所有与任务相关的规则文件。
项目约定与共享规则冲突时，先向项目所有者说明冲突并确认处理方式。
```

Claude 使用项目根目录的 `CLAUDE.md` 引用同一规则：

```md
@.agent-rules/README.md

按该文件的索引读取 `.agent-rules/rules/` 下所有与任务相关的规则；冲突先与项目所有者确认。
```

入口文件只记录项目特有约定和规则引用，不复制共享规则正文。各 Agent 的加载行为可能因版本和启动目录而不同；首次接入时检查它是否确实读取了基础规则。CI
或新设备克隆项目后，先运行 `git submodule update --init --recursive`。

## 维护与同步

1. 在本仓库修改规则并提交 PR；涉及新增架构框架、引入现成框架与手写实现的取舍等决策时，先与项目所有者讨论。
2. 合并后，各项目运行 `git submodule update --remote .agent-rules`，检查规则差异，再提交 submodule 版本指针并发起项目 PR。
3. 项目不应直接修改 submodule 中的规则来维护私有版本；项目特有要求放在自己的 `AGENTS.md` / `CLAUDE.md`，通用改动回提到本仓库。

Git 只负责版本化与固定引用，不会自动更新其他项目。每个项目何时升级规则由项目 PR 决定。
