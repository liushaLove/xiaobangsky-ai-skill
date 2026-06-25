# fs-aoi 前端知识与 Agent 协作包

本目录不是 `fs-aoi-web` 的源码仓库本体，而是围绕 fs-aoi 项目前端沉淀的一套 AI 协作资料包。它包含 4 个项目角色 Agent、若干可复用 Skill、项目技术规范、Repowiki 项目知识库，以及一次任务的任务卡片与测试报告，用于帮助 AI 在 fs-aoi 前端场景下更稳定地理解项目、拆解任务、生成代码、做测试与维护文档。

资料内容对应的目标项目是 `KCOA-KJDP / fs-aoi-web`，一个基于 Vue 3、Vite、Pinia、Vue Router 和 KJDP 企业级前端框架构建的金融综合业务服务平台。项目采用门户工作台 + 多业务模块的结构，覆盖 AOI 业务受理、UAS 账户服务、IDM 影像采集、COP 协同平台、Frame 框架页和 ISC 智能服务等模块。

> 脱敏说明：本文档中出现的环境地址均使用示例假地址，例如 `http://192.0.2.10:10080/`、`http://192.0.2.20:10081/`。这些地址来自保留示例网段或占位端口，不代表真实环境。

## 目录结构

```text
front-end/
├── agents/                         # 4 个项目 Agent 角色说明
│   ├── lead-developer.md           # 前端主程 Agent
│   ├── project-assistant.md        # 项目助理 / 协作枢纽 Agent
│   ├── tester.md                   # 测试工程师 Agent
│   └── ui-designer.md              # UI 设计师 Agent
├── skills/                         # 可复用项目技能
│   ├── project-tech-and-standards.skill.md
│   ├── kui-general-view/
│   ├── xml-to-general-view/
│   └── git-commit-aigc/
├── repowiki/zh/                    # fs-aoi-web 项目知识库
│   ├── meta/
│   └── content/
└── doc/
    └── T-20260609-001/             # 示例任务材料、任务卡片与测试报告
```

## Agent 角色

### Lead Developer

前端主程角色，负责将业务需求转化为 Vue 3 + Vite + KJDP UI 的可维护实现。重点约束包括：

- 使用 Vue 3 `<script setup>`、JavaScript、Pinia、Vue Router、SCSS。
- 统一使用 `@szkingdom.kjdp/ui` 的 `Kui*` 组件。
- 只读查询列表优先使用 `KuiGeneralView + QueryForm + TableView + Column`。
- 可编辑表格使用 `KuiTable + KuiTableColumn`。
- HTTP 请求统一走 `@szkingdom.kjdp/core` 的 `request.service` 或 `request.request`。
- 构建、Lint、版本注入和模块文档更新都纳入交付门禁。

### Project Assistant

项目助理 / 协作枢纽角色，负责需求拆解、任务编排、文档治理、版本节奏和跨角色同步。它关注：

- 将需求拆成页面、组件、接口、Store、文档等任务。
- 协调 Lead Developer、UI Designer、Tester 的交付节奏。
- 维护任务卡片、上线检查清单、风险登记、Release Notes。
- 确保 `agents/`、`skills/`、`repowiki/` 与任务文档同步更新。

### UI Designer

金融政务后台方向的界面设计角色，负责视觉一致性、交互规范和多主题体验。核心要求包括：

- 设计必须对齐 KJDP UI 组件库，不重复设计已有控件。
- 颜色、字号、圆角、间距等必须基于 `var.scss` 和主题 CSS 变量。
- 支持 default、fs25、dark、yellow 等主题。
- 关键交互控件需要完整状态集，包括默认、悬停、激活、禁用、错误、加载、空数据。
- 适老化主题需要关注字号、对比度和点击区域。

### Tester

测试工程师角色，负责功能测试、回归测试、接口验证、多主题与多模式兼容验证。测试关注点包括：

- 独立 webapp 与 KONE 子系统 iframe 模式。
- default、fs25、dark、yellow 等多主题回归。
- Chrome、Edge、定制浏览器与不同分辨率。
- 金融场景下的权限、会话、脱敏、加密链路、双重确认、表单边界、大数据分页、导出等。
- 缺陷报告必须包含模块路径、版本、主题/模式、复现步骤、接口证据、traceId、截图或录屏。

## Skills

### project-tech-and-standards

项目技术栈与开发规范总纲。它沉淀了 fs-aoi-web 的版本、技术栈、目录结构、路径别名、ESLint/Prettier、SCSS、HTTP、路由、状态管理、UI 全局配置、构建与包管理规范。

关键技术栈：

- Vue `3.4.15`
- Vue Router `4.2.5`
- Pinia `2.1.7`
- Vite `5.x`
- `@szkingdom.kjdp/core`
- `@szkingdom.kjdp/ui`
- `@szkingdom.kjdp/icons-vue`
- ECharts 6、xlsx、docxtemplater、highlight.js、VueUse 等生态依赖

示例开发代理地址在 README 中统一脱敏为：

```text
/api -> http://192.0.2.10:10080
私有 npm 源 -> http://192.0.2.20:10081/repository/npm-group/
开发访问地址 -> http://192.0.2.30:10082/
```

### kui-general-view

用于生成通用查询、列表、增删改查页面的 Skill。它覆盖：

- `KuiGeneralView`
- `QueryForm`
- `Item`
- `TableView`
- `Column`
- `TableButtons`
- `TableButton`
- `TablePagination`
- `TableDialog`

适用场景包括“通用查询页”“查询列表页”“增删改查页面”“根据查询条件和展示列生成页面”等。

### xml-to-general-view

用于将旧版 XML 视图配置转换为 Vue 3 的 `KuiGeneralView` 页面。它提供了 XML 元素到 Vue 组件的映射规则：

- `<view>` -> `<KuiGeneralView>`
- `<conf>` -> `<QueryForm>`
- `<qry_item>` -> `<Item>`
- `<btn_item>` -> `<TableButton>` 或 `<KuiButton>`
- `<col_item>` -> `<Column>`

它还沉淀了机构选择、字典、动态数据源、隐藏字段、按钮事件、编辑标志、主从表、纯表单视图等转换经验。

### git-commit-aigc

用于生成符合公司 AIGC 标记要求的 commit message。它要求在 AI 参与生成代码时标注 `aigc` 相关信息，并可按需要计算或说明 AI 采纳率。

## Repowiki 覆盖范围

`repowiki/zh/content/` 是项目知识库，内容覆盖：

- 项目概述、快速开始、项目结构、技术栈、故障排除。
- 门户系统：登录、路由、状态管理、主题、门户布局、工作台、导航栏、桌面、应用中心、标签页、iframe 容器、个性化设置等。
- 配置系统：应用配置、业务配置、KJDP 框架配置、HTTP 配置。
- AOI 业务受理系统：业务受理、复核、客户识别、工作台视图、流程处理、门户集成接口。
- 业务模块：AOI/UAS/IDM 等模块职责，代理信息综合查询、银行转账日志查询等业务案例。
- API 参考：组件 API、服务接口、Hook 接口、工具函数、`@szkingdom/kjdp-core` API。
- 插件系统：插件开发、版本资源加载器、本地目录代理。
- 构建与部署：开发服务器、生产构建、构建配置、部署指南。

这些文档可以作为 Agent 处理真实需求时的上下文来源：先读角色约束，再读项目规范，最后按模块查 Repowiki。

## 目标项目摘要

fs-aoi-web 的整体架构可概括为三层：

- 表现层：Vue 3 + KJDP UI + 多主题体系，提供统一表单、表格、弹窗、工作台、门户布局。
- 业务层：AOI、UAS、COP、IDM、ISC、Frame 等业务模块独立组织，通过门户统一接入。
- 基础设施层：KJDP Core、配置中心、服务接口映射、Vite 构建链路、本地代理与版本化资源加载器。

主要业务能力包括：

- 门户工作台、动态菜单、标签页、应用中心和个性化配置。
- AOI 业务受理与复核流程编排。
- UAS 客户识别、账户服务、代理信息综合查询。
- IDM 影像采集、签名、摄像头、视频与流程节点联动。
- 银行转账日志查询、业务监控、流程节点与服务接口映射。

## 常用项目约束

- 不随意新增 npm 依赖，新增依赖需要评审。
- 代码使用 JavaScript，不引入 TypeScript 编译链路。
- Vue SFC 使用 `<script setup>`。
- 样式使用 SCSS，颜色和尺寸优先引用变量，避免硬编码。
- 业务接口使用 service code，经统一 HTTP 配置映射到对应服务。
- 成功响应通常以 `MSG_CODE` 为 `'0'` 或 `'100'` 判定。
- 构建时通过 `APP_VERSION` 注入版本；必要时可使用 hash 构建模式。
- 提交前至少执行 lint 或 lint fix。

## 示例任务资料

`doc/T-20260609-001/` 保存了一次 AI 协作任务材料：

- `tasks/task-lead-developer.md`：开发任务卡片。
- `tasks/task-ui-designer.md`：UI 设计任务卡片。
- `tasks/task-tester.md`：测试任务卡片。
- `AI机器人测试报告.md`：V3 正式回归测试报告。
- `T-20260609-001.doc`：原始需求文档。

该任务体现了角色协作闭环：需求拆解 -> UI 设计 -> 开发实现 -> 测试验证 -> 报告归档。

## 推荐使用方式

1. 处理开发任务时，先读取 `agents/lead-developer.md` 和 `skills/project-tech-and-standards.skill.md`。
2. 如果是通用查询或增删改查页面，继续读取 `skills/kui-general-view/SKILL.md`。
3. 如果是旧 XML 页面迁移，继续读取 `skills/xml-to-general-view/SKILL.md`。
4. 如果需要理解具体业务模块，再到 `repowiki/zh/content/` 中按模块查阅。
5. 如果涉及测试、发布或协作节奏，读取 `agents/tester.md`、`agents/project-assistant.md` 和相关任务卡片。

