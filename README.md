# xiaobangsky-ai-skill

`xiaobangsky-ai-skill` 是一套面向 AI/Codex 协作的项目知识包集合，用来把真实项目中的技术事实、开发规范、角色分工、知识库和任务样例沉淀成可复用上下文。

它不是业务源码仓库，而是给 AI Agent 使用的「项目说明书 + 技能包 + 知识库」。当前内容覆盖三类场景：

- `fs-aoi/front-end`：fs-aoi-web 前端项目知识包。
- `kcop/front-end`：copweb / KCOP legacy 前端项目知识包。
- `server`：fs-aoi-biz Java 后端项目知识包。

## 目录结构

```text
xiaobangsky-ai-skill/
├── fs-aoi/
│   └── front-end/
│       ├── agents/        # 前端协作 Agent：主程、项目助理、测试、UI 设计
│       ├── skills/        # Vue/KJDP UI 前端开发技能与通用页面技能
│       ├── repowiki/zh/   # fs-aoi-web 中文项目知识库
│       └── doc/           # 示例需求、任务卡片与测试报告
├── kcop/
│   └── front-end/
│       ├── agents/        # KCOP legacy 前端协作 Agent
│       ├── skills/        # KJDP4/KUI/jQuery/Backbone 开发规范
│       ├── repowiki/zh/   # KCOP 中文项目知识库
│       └── doc/           # 示例任务资料
└── server/
    ├── agents/            # 后端主程、项目助理、测试 Agent
    ├── skills/            # Java 后端、服务注册、后端测试、提交规范技能
    └── repowiki/zh/       # fs-aoi-biz 后端知识库
```

## 三套知识包定位

### fs-aoi 前端

`fs-aoi/front-end` 面向 `fs-aoi-web`，定位为证券业务综合服务平台前端门户应用的 AI 协作资料包。

核心技术栈：

- Vue 3.4.x、Vue Router 4、Pinia、Vite 5。
- `@szkingdom.kjdp/core`、`@szkingdom.kjdp/ui`、`@szkingdom.kjdp/icons-vue`。
- SCSS、KJDP UI 主题体系、多业务模块门户。

重点技能：

- `project-tech-and-standards.skill.md`：项目技术栈、目录结构、HTTP、路由、状态、构建和代码规范。
- `kui-general-view/SKILL.md`：使用 `KuiGeneralView` 生成通用查询、列表、增删改查页面。
- `xml-to-general-view/SKILL.md`：将旧 XML 视图配置迁移为 Vue 3 + `KuiGeneralView` 页面。
- `git-commit-aigc/SKILL.md`：AI 参与代码后的提交信息规范。

适用任务包括：Vue 页面开发、KJDP UI 页面生成、旧 XML 页面迁移、门户/工作台/业务模块维护、前端测试与任务拆解。

### KCOP 前端

`kcop/front-end` 面向 `copweb / KCOP`，定位为 legacy 多页面前端项目的 AI 协作资料包。

它特别强调：KCOP 不是 Vue/Vite/Pinia 项目，而是 KJDP4/KUI 体系下的静态资源工程。

核心技术栈：

- `loader.js` + AMD/CMD 风格模块。
- KJDP4 / KUI 4.0.3。
- jQuery、Deferred、Backbone.Model、Backbone.View。
- artTemplate / tmod 模板。
- Grunt 与批处理构建脚本。

重点技能：

- `project-tech-and-standards.skill.md`：KCOP legacy 前端的技术事实、入口链路、页面规范、KUI 组件、ajax 服务和构建方式。
- `git-commit-aigc/SKILL.md`：AI 生成或辅助修改代码时的提交信息规范。

适用任务包括：老 KUI 页面维护、HTML + JS + CSS 联动修改、`data-main` 页面定位、`ajax.post` 服务调用排查、模板编译、Grunt 构建和 legacy 兼容性测试。

### fs-aoi-biz 后端

`server` 面向 `fs-aoi-biz`，定位为 AOI 业务受理 Java 后端项目的 AI 协作资料包。

核心技术栈：

- Java 8。
- Maven 多模块工程。
- Spring Boot 2.5.x。
- KJDP/Koca 体系。
- MyBatis、XML 元数据注册、ATOM/BEX/service 链路。

重点技能：

- `project-tech-and-standards.skill.md`：后端项目边界、模块职责、Maven 命令、Java 编码约定和质量门槛。
- `aoi-java-backend/SKILL.md`：Java 后端功能开发、缺陷修复、模块导航和代码评审。
- `aoi-service-registry/SKILL.md`：KJDP 服务码、business_code、ATOM、BEX、MyBatis 链路追踪。
- `aoi-backend-test/SKILL.md`：后端测试、mock 数据、`fs-aoi-test` 和 Surefire 配置注意事项。
- `git-commit-aigc/SKILL.md`：AI 参与提交的 message 规范。

适用任务包括：服务码链路排查、ATOM/BEX 注册维护、MyBatis SQL 修改、Java 业务逻辑开发、Maven 构建、后端测试和发布风险说明。

## Agent 角色体系

每套知识包都围绕 AI 协作拆分了角色：

- `lead-developer`：负责技术方案、代码实现、变更影响分析和代码评审。
- `project-assistant`：负责需求拆解、任务编排、风险登记、交付物整理和文档同步。
- `tester`：负责测试用例、回归范围、接口验证、缺陷复现和测试报告。
- `ui-designer`：前端包中包含该角色，负责 KJDP/KUI 体系下的界面布局、组件选型和视觉一致性。

这些 Agent 文档的作用不是替代人，而是让 AI 在处理任务前先站到正确的项目角色和技术边界里，减少把前端当后端、把 legacy 当 Vue、把配置链路当普通 REST 接口等误判。

## Repowiki 知识库

`repowiki/zh/content/` 是按项目沉淀的中文知识库。内容覆盖：

- 项目概述、快速开始、项目结构、技术栈。
- 门户系统、工作台、路由、状态管理、主题系统。
- 配置系统、HTTP 配置、业务配置、KJDP 框架配置。
- 插件系统、版本资源加载器、本地目录代理。
- API 参考、组件 API、服务接口、工具函数。
- 业务模块、客户识别、业务受理、流程处理、门户集成。
- 构建、部署、调试、测试策略、性能优化和故障排除。

使用 AI 处理任务时，推荐先读对应 Agent，再读项目规范 Skill，最后按模块查 Repowiki。

## 推荐使用顺序

处理 fs-aoi 前端任务：

1. 读 `fs-aoi/front-end/agents/lead-developer.md` 或对应角色 Agent。
2. 读 `fs-aoi/front-end/skills/project-tech-and-standards.skill.md`。
3. 通用页面读 `kui-general-view/SKILL.md`，XML 迁移读 `xml-to-general-view/SKILL.md`。
4. 按业务模块查 `fs-aoi/front-end/repowiki/zh/content/`。

处理 KCOP 前端任务：

1. 读 `kcop/front-end/skills/project-tech-and-standards.skill.md`。
2. 从页面标题、字段、服务号或 HTML 的 `data-main` 定位模块。
3. 沿 HTML、`src/*.js`、CSS、service、template、构建脚本一起检查。
4. 按模块查 `kcop/front-end/repowiki/zh/content/`。

处理 fs-aoi-biz 后端任务：

1. 读 `server/skills/project-tech-and-standards.skill.md`。
2. 开发任务读 `server/skills/aoi-java-backend/SKILL.md`。
3. 服务链路任务读 `server/skills/aoi-service-registry/SKILL.md`。
4. 测试任务读 `server/skills/aoi-backend-test/SKILL.md`。
5. 按服务码、ATOM、BEX、mapper、Java 类和测试数据追完整链路。

## 脱敏约定

本 README 不记录真实环境地址、真实 IP、真实端口、账号、密码、token、数据库连接串或内部域名。

如果需要在文档中说明地址，请统一使用保留示例地址和假端口，例如：

| 类型 | 示例写法 |
| --- | --- |
| 前端访问地址 | `http://192.0.2.10:18080/` |
| 后端代理地址 | `http://192.0.2.20:19090/` |
| npm 私服示例 | `http://192.0.2.30:18081/repository/npm-group/` |
| 本地开发示例 | `http://127.0.0.1:18080/` |

说明：

- `192.0.2.0/24`、`198.51.100.0/24`、`203.0.113.0/24` 是文档示例网段，不应指向真实环境。
- 文档中出现类似 `http://ip:port/`、`http://ip1:port1/` 的内容，应视为已脱敏占位。
- 新增 README、任务卡片、测试报告或知识库内容时，不要写入真实 IP、真实端口、真实账号、真实 token 或真实生产环境信息。

## 维护建议

- 三套知识包的技术边界要分开维护：fs-aoi 前端是 Vue/Vite，KCOP 前端是 legacy KJDP4/KUI，server 是 Java 后端。
- 更新业务规则时，同步更新 Agent、Skill 和 Repowiki，避免 AI 读到过期上下文。
- 示例任务和测试报告可以保留为协作样板，但不要把历史样例里的技术口径直接套用到其他项目。
- 提交由 AI 生成或辅助修改的代码时，按对应 `git-commit-aigc` Skill 标注 `AIGC`。
