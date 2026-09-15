# Agent Rules

为 Codex、Claude、OpenCode、Pi 等编码 Agent 维护的共享工程规范。仓库内的规则是唯一原稿；各项目通过 Git 引用固定版本，避免在不同 Agent 的入口文件里重复维护规则正文。

## 规则文件

| 文件 | 内容 |
| --- | --- |
| [`base-rule.md`](base-rule.md) | 通用基础规则：技术栈、前后端架构、契约、中间件、可观测性、质量门控和决策要求。 |

后续可新增 `frontend-rule.md`、`backend-rule.md`、`testing-rule.md` 等专题规则，并在这里登记用途。专题规则补充基础规则；若与基础规则或项目现有约定冲突，应先说明取舍并与项目所有者确认，不得静默覆盖。

## 在项目中引用

推荐将本仓库作为项目的 Git submodule 引入，使每个项目固定规则版本，规则更新通过项目自身的 PR 审核：

```bash
git submodule add -b master https://github.com/spcookie/agent-rules.git .agent-rules
git commit -m "Add shared agent rules"
```

在项目根目录的 `AGENTS.md` 中加入以下入口，供 Codex、OpenCode、Pi 使用：

```md
# Project instructions

开始任务前，读取并遵守 `.agent-rules/base-rule.md`。
如果任务涉及某个专题，也读取 `.agent-rules/` 下相关专题规则。
项目约定与共享规则冲突时，先向项目所有者说明冲突并确认处理方式。
```

Claude 使用项目根目录的 `CLAUDE.md` 引用同一规则：

```md
@.agent-rules/base-rule.md

涉及专题时也读取 `.agent-rules/` 下相关规则；冲突先与项目所有者确认。
```

入口文件只记录项目特有约定和规则引用，不复制共享规则正文。各 Agent 的加载行为可能因版本和启动目录而不同；首次接入时检查它是否确实读取了基础规则。CI 或新设备克隆项目后，先运行 `git submodule update --init --recursive`。

## 维护与同步

1. 在本仓库修改规则并提交 PR；涉及新增架构框架、引入现成框架与手写实现的取舍等决策时，先与项目所有者讨论。
2. 合并后，各项目运行 `git submodule update --remote .agent-rules`，检查规则差异，再提交 submodule 版本指针并发起项目 PR。
3. 项目不应直接修改 submodule 中的规则来维护私有版本；项目特有要求放在自己的 `AGENTS.md` / `CLAUDE.md`，通用改动回提到本仓库。

Git 只负责版本化与固定引用，不会自动更新其他项目。每个项目何时升级规则由项目 PR 决定。
