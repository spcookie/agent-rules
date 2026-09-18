# 质量、测试与交付

## Lint、格式化与质量门控

- TypeScript/React：使用 ESLint + typescript-eslint 做代码静态检查，并启用 React Hooks 相关规则；使用 Prettier 做格式化，`eslint-config-prettier` 避免两套工具争夺格式规则。`tsc --noEmit`（或项目的 `tsc -b`）单独负责类型检查，不能以 ESLint 代替类型检查。
- Kotlin：使用 ktlint 统一格式与基础风格检查；Gradle 提供 `ktlintCheck` 和 `ktlintFormat`，CI 只运行检查任务，不自动改写源码。需要更深入的复杂度或潜在缺陷分析时，先与我决策是否引入 detekt 等工具及具体规则集。
- 格式、lint、类型检查规则提交到仓库，开发机与 CI 使用相同配置和锁定的工具版本。生成代码（如 Proto 生成物）排除在手写代码的格式/lint 范围之外；规则抑制须局部、注明原因，不得为通过门控大范围关闭规则。
- CI 的必过门控：格式检查、lint、TypeScript 类型检查、Kotlin 编译、受影响测试及构建；涉及 HTTP API 时运行 Spectral、openapi-diff，涉及 gRPC/Protobuf 时运行 Buf，涉及 JSON 消息时运行 JSON Schema/ajv-cli 校验（有 AsyncAPI 时追加 AsyncAPI CLI），涉及数据库时验证 Flyway 迁移；涉及多语言时校验各语言 key 集合与占位符一致（至少覆盖 `zh-CN` 与 `en`）。门控失败不得合并，不用仅显示警告或自动修复掩盖失败。
- 测试覆盖核心业务规则及本次变更的主要成功/失败路径；覆盖率门槛和安全/依赖扫描如需设为硬门控，先确定项目基线、误报处理与维护责任，再与你决策。

## 测试与交付

- 前端：纯 `model` 函数做单元测试；Query/Store 逻辑及关键页面状态做集成或组件测试；核心用户流程按需要做端到端测试。涉及多语言或主题时，非默认语言和两套主题的渲染纳入同一批测试。
- 后端：领域规则与应用用例优先做无需中间件的测试；Exposed、Redis、RabbitMQ 等适配器用集成测试验证映射、事务与失败路径。
- 提交前运行相关项目的格式检查、lint、类型检查、构建及受影响测试。新增接口、消息或持久化结构时同步更新 OpenAPI/Proto/消息契约、Flyway 迁移和相关测试。
- 不做与需求无关的大范围重构；若必须偏离本规范，在代码或变更说明中记录理由与后续处理方式。
