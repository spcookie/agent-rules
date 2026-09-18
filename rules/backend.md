# 后端规则

## 技术栈

- Gradle Kotlin DSL：构建脚本只用 `.kts`（`build.gradle.kts`、`settings.gradle.kts`），不新增 Groovy DSL；依赖与插件版本集中在 version catalog（`gradle/libs.versions.toml`），模块内不重复写版本号。统一用 wrapper（`./gradlew`）构建，wrapper 提交到仓库，CI 与本地使用同一版本。
- Kotlin + Ktor：Ktor 负责 HTTP 入口、路由、认证、序列化与基础设施装配；路由不直接编写业务规则或 SQL。
- Project Reactor、kotlinx-coroutines-reactor：需要响应式编程时统一使用 Reactor；Reactor 与 Kotlin 协程互操作使用 `kotlinx-coroutines-reactor`，不在领域层暴露 `Mono`、`Flux` 等框架类型。
- Koin：负责依赖注入与应用启动时的装配；领域模型和领域规则不依赖 Koin 注解/API。
- Exposed（ORM、JDBC）：负责 PostgreSQL 持久化适配器与事务实现；数据库实体和 Exposed 类型不越过持久化边界进入领域层。
- Flyway：管理 PostgreSQL 的版本化 schema 迁移；迁移脚本随代码提交，禁止以 Exposed 自动建表或手工改生产库代替迁移。
- kotlinx.serialization JSON、Jackson：作为 JSON 序列化方案。每个服务明确一个默认方案；需要兼容第三方接口或特殊格式时才在指定边界使用另一个，不让两套注解/配置同时决定同一 DTO 的格式。
- Caffeine：作为进程内本地缓存方案；缓存实现位于输出适配器，不让领域规则依赖具体缓存 API。
- JobRunr：负责后台任务的调度和执行；任务入口调用应用层用例，业务规则不依赖 JobRunr。
- OpenTelemetry：作为后端追踪、指标与日志关联的统一可观测性标准；导出端点及采样策略通过环境配置，不把具体观测平台写入业务代码。
- Apache Commons：仅在确有必要时使用成熟工具能力；不为简单语言/标准库操作引入额外依赖。
- Arrow：用于确实受益于函数式组合、显式错误或不可变建模的业务流程；不要求所有代码函数式化，也不把 Arrow 类型强加给 HTTP/数据库外部契约。
- 国际化资源：服务端文案默认用 ICU MessageFormat（ICU4J）风格的按语言 bundle 组织，需要提供 `zh-CN` 和 `en` 两套（语言范围见 [`frontend.md`](frontend.md) 的 i18n 规则），不在代码中硬编码面向用户的文本；引入或更换 i18n 库前按 [`general.md`](general.md) 与我确认。

## 架构：六边形 / 洋葱架构

依赖方向始终向内：`adapters -> application -> domain`。领域层不依赖应用层，应用层不依赖具体适配器。按业务域划分包或模块，并在每个业务域中保持以下职责：

```text
domain/          # 实体、值对象、领域服务、领域错误与不变量
application/     # 用例、命令/查询、事务边界、输出端口
adapters/in/     # Ktor HTTP、消息消费者、JobRunr 任务入口
adapters/out/    # Exposed/JDBC、Redis、RabbitMQ、外部 API 实现
bootstrap/       # Koin 装配、Ktor 配置、启动逻辑
```

- `domain` 用纯 Kotlin 表达业务规则，尽量让规则可在无框架、无数据库环境下测试。
- `application` 编排领域对象与端口，定义用例输入/输出和业务事务范围；端口由需求方定义，具体实现留在外层。
- 输入适配器完成协议解析、认证/授权入口与输入校验，然后调用用例；HTTP DTO、消息格式不直接作为领域模型。
- 输出适配器实现持久化、缓存、消息发布及第三方调用；在边界映射领域模型与存储/传输模型。
- 用 Koin 在 `bootstrap` 绑定端口与实现。不要用 service locator 在业务代码中动态取依赖。
- 事务由应用用例或其事务端口协调；不要跨长时间网络调用保持数据库事务。需要数据库写入与消息发布一致时，明确选择 outbox 等一致性策略，不假设二者天然原子。
- 可采用单模块按包分层或多模块实现；只有实际存在独立演进、复用或依赖隔离需求时才拆 Gradle 模块。
