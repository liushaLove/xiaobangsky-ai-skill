# KCOP 前端 AI Skill 知识包

本目录是一套面向 `copweb / KCOP` legacy 前端项目的 AI 协作知识包。它不是业务代码仓本体，而是把项目技术事实、开发规范、知识库、AI Agent 角色和任务样例整理成可复用的上下文，用于帮助 AI 在处理 KCOP 前端需求时保持一致判断。

核心定位：让 AI 明确当前 KCOP 前端是 KJDP4/KUI 体系下的多页面 legacy 静态资源工程，而不是 Vue、Vite、Pinia、Vue Router 或 TypeScript 工程。

## 目录结构

```text
front-end/
├─ agents/                         # 4 个 AI Agent 角色定义
│  ├─ project-assistant.md          # 项目助理：任务编排、风险、文档治理
│  ├─ lead-developer.md             # 主力开发：方案、实现、代码规范
│  ├─ ui-designer.md                # UI 设计：KUI 组件、布局、视觉走查
│  └─ tester.md                     # 测试：用例、回归、缺陷、测试报告
├─ skills/                          # 可复用技能和项目规范
│  ├─ project-tech-and-standards.skill.md
│  └─ git-commit-aigc/SKILL.md
├─ repowiki/zh/                     # KCOP 前端中文知识库
│  ├─ meta/repowiki-metadata.json
│  └─ content/
└─ doc/T-20260609-001/              # 一次 AI 协作任务样例与测试报告
```

## 项目事实基线

知识包围绕 `copweb / KCOP` 前端项目整理，项目形态如下：

| 维度 | 当前项目事实 |
| --- | --- |
| 应用形态 | 多 HTML 页面 + `loader.js` 动态模块加载 |
| 模块系统 | AMD/CMD 风格 `define(function(require, exports, module){ ... })` |
| UI 体系 | KJDP4 / KUI 4.0.3 jQuery 插件 |
| DOM 与异步 | jQuery + Deferred |
| 页面组织 | Backbone.Model / Backbone.View + 页面级 `$ready` |
| 请求封装 | 全局 `ajax`、`ajax.post({ req })`、公共 service |
| 模板 | artTemplate / tmod |
| 构建 | Grunt + `build*.bat` |
| 主业务目录 | `apps/opp/**` |

必须避免把该项目按 Vue/Vite 现代前端工程处理。常见错误包括：默认寻找 `src/main.js`、`vite.config.js`、Vue Router、Pinia、`request.service`、`<script setup>` 或 npm dev server。

## 4 个 AI Agent

### Project Assistant

项目助理负责需求接收、范围澄清、任务派发、风险登记、文档治理和上线检查。它的价值是把 Lead Developer、UI Designer、Tester 拉到同一套 KCOP legacy 前端事实基线上，避免沿用旧的 Vue/Vite 口径。

适合处理：

- 任务拆解和排期。
- 页面路径、字段、服务号、风险项整理。
- 发布检查清单。
- 文档差异记录和归档。

### Lead Developer

主力开发负责技术方案、代码实现、代码评审和开发规范维护。它强调从页面 HTML 的 `data-main` 定位主模块，沿用 KUI、jQuery、Backbone、Grunt、tmod 等既有体系。

适合处理：

- HTML / JS / CSS / service / template 改动方案。
- `flow.init` 流程页、Backbone 页面、KUI 表单表格页面维护。
- loader、require 路径、ajax 请求、字段链路排查。
- 构建和模板编译影响判断。

### UI Designer

UI Designer 负责 legacy KUI 体系下的页面布局、组件选型、视觉走查和样式风险控制。它不输出脱离当前工程的 Vue 组件或现代大屏式设计，而是优先复用 KUI 插件和现有主题。

适合处理：

- `kui-textinput`、`kui-combobox`、`kui-combotree`、`datagrid`、`dialog` 等组件选型。
- 表单 label、输入框、按钮、表格、弹窗布局建议。
- 固定宽度后台页面的可读性和操作效率检查。
- 页面级 CSS 与公共主题影响面评估。

### Tester

Tester 负责功能测试、回归测试、接口验证、页面兼容性验证和发布测试报告。测试方式以手工业务验证、浏览器 DevTools、Network 证据和构建产物检查为主。

适合处理：

- 功能、边界、异常、权限、接口测试用例。
- service、bex_codes、入参、MSG_CODE、MSG_TEXT、traceId 证据收集。
- 空数据、会话超时、接口失败、KUI 组件状态验证。
- 发布风险结论和回归矩阵。

## 推荐阅读顺序

新接手时建议按下面顺序读取：

1. `agents/project-assistant.md`：理解整体协作方式。
2. `skills/project-tech-and-standards.skill.md`：掌握最高优先级技术基线。
3. `repowiki/zh/content/知识库导航.md`：进入知识库索引。
4. `repowiki/zh/content/项目概述.md`：理解项目定位和运行链路。
5. `repowiki/zh/content/技术栈.md`：确认当前技术栈和禁用口径。
6. `repowiki/zh/content/快速开始.md`：学习页面定位、修改和构建路径。
7. `repowiki/zh/content/kjdp-core-api.md`：掌握全局对象、ajax 和 KUI API。
8. `repowiki/zh/content/API参考/组件API.md` 与 `API参考/服务接口.md`：处理业务页时高频使用。
9. `repowiki/zh/content/构建与部署/构建配置.md`：了解 Grunt、tmod、dist 与发布目录。
10. `repowiki/zh/content/故障排除.md`：排查页面空白、组件失效、请求失败、模板不更新和构建失败。

## 典型工作流

处理一个 KCOP 前端需求时，可按下面方式协作：

1. Project Assistant 记录任务 ID、模块、页面路径、字段名、服务号、验收标准和风险。
2. Lead Developer 读取技术基线和相关 repowiki，定位 HTML 的 `data-main` 和对应 `src/*.js`。
3. UI Designer 确认 KUI 组件、表单布局、表格列、弹窗尺寸、空数据和禁用状态。
4. Lead Developer 按 legacy 规范实现，不引入新框架，不批量格式化历史文件。
5. Tester 根据改动范围输出测试用例，验证页面加载、KUI 组件、请求、权限、流程页、iframe/标签页和构建产物。
6. Project Assistant 汇总发布检查、缺陷状态、未验证风险和文档同步项。

## 开发定位口诀

```text
页面标题/字段/服务号 -> 搜索 HTML -> 看 data-main -> 找 src/*.js
-> 读 exports.$ready / flow.init / Backbone.View
-> 查 KUI 组件 id/name 和 JS 选择器
-> 查 ajax.post / service / bex_codes
-> 如涉及模板，重新生成 apps/tpls/src/template.js
```

常用命令：

```powershell
rg -n "页面标题|字段名|服务号|data-main" apps basic frame
rg --files | rg "页面名"
build-tpls.bat
build.bat
```

## 脱敏约定

本文档不输出真实环境地址。涉及平台地址、后端代理地址、接口地址时，统一使用文档专用假地址：

| 类型 | 假地址示例 |
| --- | --- |
| 前端平台地址 | `http://192.0.2.10:18080/` |
| 后端代理地址 | `http://192.0.2.20:19090` |
| 本地示例地址 | `http://127.0.0.1:18080/` |

说明：

- `192.0.2.0/24` 是文档示例网段，可用于占位，不应指向真实环境。
- 原始文档中如出现 `http://ip:port/`、`http://ip1:port1` 等地址，应视为已脱敏占位。
- 写新任务卡片、测试报告或 README 时，不要填真实 IP、真实端口、真实账号密码、真实 token。

## 文档状态说明

`repowiki/zh/content/` 已经以 KCOP legacy KJDP/KUI 项目重新梳理，主线口径比较稳定。`doc/T-20260609-001/` 是一次历史 AI 协作样例，部分内容仍保留了 fs-aoi-web / Vue / Vite 口径，且若干文件存在编码显示异常；使用时建议把它作为“任务样例”和“测试报告样例”，不要直接作为当前 KCOP 技术基线。

`skills/git-commit-aigc/SKILL.md` 是提交信息规范技能，当前文件也存在编码显示异常。可识别的核心要求是：AI 生成代码提交时，commit message 需要包含大写 `AIGC` 标识，并在需要时标注采纳率。

## 维护红线

- 不要把 KCOP 当成 Vue/Vite/Pinia 项目。
- 不要默认引入 TypeScript、ESM、`async/await`、可选链、空值合并等可能破坏兼容的写法。
- 不要随意修改 `loader.js`、`loader-config.js`、`loader-frame-config.js`、`frame/core/ajax.js`、`basic/ready.js`、`frame/kui.js`、`lib/kui/**` 等公共基础层。
- 不要只改 HTML 字段而不检查 JS 选择器、form load/save、后端字段和 CSS。
- 不要修改 `apps/tpls/**/*.html` 后忘记重新生成 `apps/tpls/src/template.js`。
- 不要在文档、任务卡片或报告中写入真实 IP、端口、账号、密码或生产环境信息。

## 适用场景

这套知识包适合用于：

- 给 AI Agent 注入 KCOP 前端项目上下文。
- 让开发、设计、测试角色围绕同一份项目事实协作。
- 快速定位 legacy KJDP/KUI 页面维护方式。
- 生成任务卡片、测试用例、风险清单、发布检查清单。
- 纠正旧文档中残留的 Vue/Vite/fs-aoi-web 口径。

