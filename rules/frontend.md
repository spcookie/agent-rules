# 前端规则

## 技术栈

- pnpm：作为唯一包管理器，提交 `pnpm-lock.yaml`，同一仓库不混用 npm/yarn，构建脚本经 `pnpm run` 执行。多包仓库使用 pnpm workspace，跨包依赖用 `workspace:` 协议、安装用 `pnpm add --filter <pkg>`；CI 与部署一律 `pnpm install --frozen-lockfile`，版本冲突用 `pnpm.overrides` 统一，不靠删锁文件或 `latest` 绕过。
- React + TypeScript：使用函数组件与 Hooks；定义明确的 props 和返回类型，避免 `any`、非必要的类型断言和组件中的隐式副作用。
- ReactUse（`@reactuses/core`）：浏览器 API、传感器、生命周期和常见交互等通用 Hook 优先复用 ReactUse；按需导入并确认 SSR 兼容性，不为简单的 `useState`/`useEffect` 包装引入额外抽象，也不以其替代 TanStack Query 的服务端状态管理。
- TanStack Router：管理路由、嵌套路由、参数解析与页面级加载；路由文件主要负责路由声明、权限入口和页面组合。
- TanStack Query：负责服务端数据的获取、缓存、失效、后台刷新及异步请求状态；mutation 成功后按 query key 精准失效或更新缓存。
- Zustand：只管理跨组件共享的客户端状态，例如临时 UI 偏好、选中项和本地工作流状态；不复制 Query 已管理的服务端数据。
- Zod：校验表单、URL 参数及不可信的 API 响应；从 schema 推导类型，避免同一契约维护两套手写类型。
- shadcn/ui：作为基础 UI 组件来源；业务组件在其上组合，避免直接在基础组件中混入业务逻辑。
- Tailwind CSS：用于常规布局、间距和视觉样式；Less 用于复杂局部样式、需要嵌套或变量组织的场景。两者共用设计 token，不对同一个组件的同一属性建立互相冲突的样式来源。
- i18n：默认用 i18next + react-i18next 组织文案，需要支持简体中文（`zh-CN`）和英文（`en`），回退链为 `zh-CN` → `en`。面向用户的文案一律走资源文件，不在 JSX、常量或校验消息里写死。key 按功能分组且语义稳定，用带占位符的整句配合 ICU 复数与分支选择，不拼接句子片段；日期、数字、货币用 `Intl.*` 格式化。当前语言来自 URL 或用户偏好，Zustand 只保存选择，不复制翻译资源；缺失 key 在开发环境显式暴露，资源按命名空间按需加载。
- a11y：以 WCAG 2.1 AA 为基线。使用语义化 HTML 和原生交互元素，不用 `div` 模拟控件；交互元素可键盘到达且焦点可见，模态框管理焦点陷阱与恢复；图标按钮有可读名称，错误与控件用 `aria-describedby`/`aria-invalid` 关联；正文对比度不低于 4.5:1，状态不只用颜色区分。优先复用 shadcn/ui（Radix）已有的无障碍行为，不为了样式换成无键盘支持的自研实现。
- 主题：颜色只通过设计 token 的语义变量暴露，组件内不写死色值；通过根元素 class 切换，Tailwind 用 `darkMode: 'class'`。默认跟随系统并允许用户覆盖，偏好与解析结果分开存放；主题在首屏渲染前确定以避免闪烁；新增颜色必须同时定义两套主题。

## 架构：按功能组织，数据与展示分离

推荐按业务功能组织代码，在功能内部区分数据、状态和展示；这是一种轻量的 feature-based 分层，不要求每个功能都建立全部目录：

```text
src/
  app/                 # 应用入口、Provider、全局配置
  routes/              # 路由声明、页面入口、布局路由
  features/
    orders/
      api/             # 请求函数、Zod 响应契约、query key/options
      model/           # 纯业务计算、类型、必要的本地 Zustand store
      hooks/           # 将请求/状态组合成页面可用的数据逻辑
      components/      # 业务展示组件
      pages/           # 页面组装：布局、状态分支、交互接线
      styles/          # 必要的功能级 Less 样式
  shared/
    ui/                # 跨功能通用组件及 shadcn/ui 基础组件
    layout/            # AppShell、导航、页面容器等复用布局
    api/               # HTTP 客户端、认证与通用错误处理
    lib/               # 无业务归属的纯函数和工具
    styles/            # 设计 token、全局样式
```

- `api` 处理传输协议和数据验证，不在展示组件中直接发请求。Query key 由所属功能集中定义，避免散落的字符串。
- `model` 放与 React 无关、可独立测试的业务规则；需要共享的客户端状态才建立 Zustand store。
- `hooks` 编排 Query、store 和事件处理。不要让 Hook 成为包含大量渲染代码的第二个页面。
- `components` 优先通过 props 接收数据与回调；可复用的展示组件不直接依赖路由、全局 store 或具体 API。
- `pages` 负责页面组装、加载/空/错误状态与布局，不承载复杂数据转换或业务判断。
- `shared` 只存放真正跨功能复用的内容；一个业务功能不得直接依赖另一功能的内部文件，需要复用时通过明确的公共接口或上移共享能力。
- 优先抽出复用布局、设计 token 和稳定组件；一次性 JSX 不必提前抽象。Less 尽量局部作用域，避免全局选择器影响其他页面。
- 避免同时把同一份数据放进组件 state、Zustand 和 Query。表单编辑态留在表单/组件，服务端状态留在 Query，跨页面客户端状态才进入 Zustand。
