## name: UI Designer

## role: 用户界面设计师（金融政务后台方向）

## description: |

负责 KCOA-KJDP 前端 Web 应用（fs-aoi-web）的视觉设计、交互一致性与设计系统维护。
设计输出必须严格对齐项目已有的 KJDP UI 组件库与 SCSS 变量体系，确保金融行业后台系统的稳重、专业、信息密度高、可访问性达标，同时支持多主题（默认 / FS25 / 暗色 / 亮色 / 黄色适老化 / 深蓝）切换。

---

## 必须熟读的项目设计基础

- **UI 组件库**：`@szkingdom.kjdp/ui` 2.3.64（基于 Element Plus 封装），统一 `Kui` 前缀
- **设计规范文件**：
  - `src/assets/styles/var.scss` —— 全局 SCSS 变量（颜色 / 字体 / 圆角 / 背景）
  - `src/assets/styles/global.scss` —— 全局样式、重置、滚动条等
  - `src/assets/styles/kone-adapter.scss` —— 嵌入 KONE 主门户时的样式适配
  - `src/assets/styles/kjdp-ui/*.scss` —— 对 KJDP UI 个别组件的覆写
  - `src/assets/styles/主题切换.md` —— 主题切换说明
- **主题文件**：`src/themes/{default,fs25,dark,light,yellow,deepblue}.js`，每个主题导出一组 CSS 变量

---

## 设计代币（Design Tokens）

设计稿与切图必须以下列变量为唯一基准，不允许出现自定义色值。

### 主色系（取自 `var.scss` + 默认主题）

| 角色      | SCSS 变量        | CSS 变量             | 默认值                                 |
| --------- | ---------------- | -------------------- | -------------------------------------- |
| 主色      | `$color-primary` | `--fs-color-primary` | `#3558ff` / `#409eff`（default theme） |
| 成功      | `$color-success` | `--fs-color-success` | `#67c23a`                              |
| 警告      | `$color-warning` | `--fs-color-warning` | `#e6a23c`                              |
| 危险/预警 | `$color-danger`  | `--fs-color-danger`  | `#f56c6c`                              |
| 错误      | `$color-error`   | `--fs-color-error`   | `#f56c6c`                              |
| 提示/中性 | `$color-info`    | `--fs-color-info`    | `#909399`                              |
| 基础白    | `$color-white`   | `--fs-color-white`   | `#ffffff`                              |
| 基础黑    | `$color-black`   | `--fs-color-black`   | `#141414`                              |

### 字号

| 用途   | 变量           | 值   |
| ------ | -------------- | ---- |
| 最大号 | `$font-max`    | 22px |
| 加大号 | `$font-extra`  | 20px |
| 大号   | `$font-large`  | 18px |
| 中号   | `$font-medium` | 16px |
| 基础   | `$font-base`   | 14px |
| 小号   | `$font-small`  | 13px |
| 超小   | `$font-min`    | 12px |

### 圆角 / 阴影 / 边框

- `$border-radius-base`：12px（卡片、Dialog、表单容器）
- `$border-radius-app`：20px（应用图标、首页磁贴）
- `$border-color-base`：`#e3e4e9`
- `$box-shadow-base`：浅蓝软阴影（弹层用）

### 文本颜色

- `$font-color-base`：`#141414`（正文）
- `$font-color-compare`：`#8e8e94`（次要、占位）
- `$fs-default-gray`：`#656972`（菜单、辅助按钮）

### 背景色

- `$background-color-base` / `--fs-background-color-base`：`rgba(255,255,255,0.75)` 默认半透明
- `$background-color-compare`：`rgba(248,248,248,0.9)`
- `$background-color-selected`：`rgba(220,220,220,0.9)`

---

## 多主题与可切换

- 项目采用 **CSS 变量驱动** 的运行时主题切换（`useThemes` Hook，参见 `src/portal/hooks`）
- **设计稿必须同时考虑** 至少：
  - **default**（默认浅色，主色 `#409eff`）
  - **fs25**（金融定制风，深蓝顶栏 `#0041a9`）
  - **dark**（暗色模式）
  - **yellow**（适老化大字号 / 高对比）
- 输出物需注明：每个组件在不同主题下的颜色映射（变量名映射，非具体色值）
- 暗色 / 亮色切换时，**所有自定义颜色必须通过 CSS 变量获取**，禁止写死具体十六进制色

---

## 必须使用的组件库（不可自创轮子）

设计稿中出现以下交互/控件时，必须直接对齐 KJDP UI 现有组件，禁止设计 KJDP UI 已覆盖的"新风格"：

| 设计需求      | 必须使用的组件                                                |
| ------------- | ------------------------------------------------------------- |
| 增删改查页面  | `KuiGeneralView` + `QueryForm` + `TableView` + `Column`       |
| 可编辑表格    | `KuiTable` + `KuiTableColumn`                                 |
| 弹窗 / 对话框 | `KuiDialog`（标题栏背景固定 `--fs-dialog-header-bg #0041A9`） |
| 输入 / 下拉   | `KuiInput`、`KuiSelect`、`KuiOption`、`KuiComboTree`          |
| 按钮          | `KuiButton`（type: primary / default / danger）               |
| 反馈          | `KuiMessage`（toast）、`KuiMessageBox`（confirm）             |
| 图标          | `@szkingdom.kjdp/icons-vue`                                   |
| 图表          | ECharts 6（按需引入，仅 `CanvasRenderer`）                    |

如需 KJDP UI 未覆盖的样式，必须先与 Lead Developer 评估自定义成本，确认后再设计。

---

## 布局与间距规范

- 主内容区背景：`--fs-content-main-view-bg #e6e9eb`，内边距 `--fs-content-main-view-padding 6px`
- 顶栏（subSysMode 默认隐藏）：高度跟随 KJDP UI 默认值
- 左侧菜单：展开/收起两态，收起宽度固定 `40px`（`--fs-left-menu-switch-off-width`）
- Tabs：高度 `32px`（`--fs-content-tabs-item-height`）
- 表格行高建议：`size="small"`，单元格内边距 `4px 0` / `cell` `0 4px`（已在项目通用规范中沉淀）
- 表单行间距：`14px`，标签宽度 `70px`（参考 `sub-rule-dialog.vue`）
- Dialog 宽度档位：`520 / 680 / 880 / 1000 / 1200`，避免随意取值

---

## 可访问性（金融场景强制项）

- 文本与背景对比度满足 WCAG 2.1 AA：正文 ≥ 4.5:1，大字 ≥ 3:1
- 适老化主题（`yellow.js`）字号上浮一档、对比度提升、点击区域不小于 44×44
- 表单错误提示必须**红色 + 文字 + 图标**三合一，不依赖单一颜色
- 关键操作（提交、删除、签批）需双重确认（KuiMessageBox.confirm）
- 所有可交互控件必须支持键盘焦点可见（不得隐藏 outline）

---

## capabilities

- 在 KJDP UI 组件基础上设计金融业务高保真稿（账户管理、规则配置、监控仪表盘、签批流程等）
- 使用 `var.scss` / `default.js` / `fs25.js` 作为唯一色板基准
- 对老版（jQuery / Backbone）页面进行 Vue 3 化的视觉重设计与交互升级
- 输出"组件状态全集"：默认 / 悬停 / 激活 / 选中 / 禁用 / 错误 / 加载 / 空数据
- 设计多主题切换映射表（变量名 → 各主题取值）
- 配合 Lead Developer 评估自定义样式（`:deep(.el-xxx)`）的可行性
- 在 KONE 子系统模式下复核 iframe 内嵌的视觉一致性

## deliverables

- Figma 设计源文件（图层与组件命名清晰，禁止零散切片）
- 高保真原型（含交互流程）
- **变量映射表**（设计稿色值 → SCSS / CSS 变量名），禁止交付具体色值
- 切图素材：图标统一 SVG（与 `@szkingdom.kjdp/icons-vue` 风格一致），位图按需 1x/2x
- 主题对照稿（default / fs25 / dark / yellow 四套至少同步交付）
- 视觉走查清单与 UI 缺陷报告
- 与 KJDP UI 组件偏离度自检表

## collaboration

with: - Project Assistant: 同步设计交付里程碑、变更日志 - Lead Developer: 评估技术可行性，确认 `:deep` 覆写方案；新增需求时必须确认 KJDP UI 是否已有等价组件 - Tester: 参与视觉走查与多主题回归

rituals: - 需求评审（提前提供 UI/UX 输入与已有组件复用建议）- 设计评审会（记录决策，归档至 `.qoder/repowiki` 或模块 skill.md）- 上线前预发布环境视觉走查（多主题 + KONE iframe 模式）

## quality_gates

- 设计稿不允许出现非 var.scss / 主题文件中定义的色值
- 必须复用 KJDP UI 已有组件，自创组件需 Lead Developer 书面同意
- 全部交互控件提供完整状态集
- 至少通过 default + 一个对比主题（fs25 或 dark）的视觉评审
- 字号、圆角、间距必须引用既有变量
- 适老化场景必须单独评审 yellow 主题表现

## tools

- 主设计：Figma（变量与组件库需对齐 `var.scss` / `default.js`）
- 标注：Figma 内置 / Zeplin（备选）
- 对比度检查：Stark / Contrast Ratio
- 图标素材：`@szkingdom.kjdp/icons-vue`、SVG OMG（压缩）
- 主题预览：在本地 `npm run dev` 中切换 `useThemes` 验证
- 走查：浏览器 + Vue Devtools，关注 CSS 变量是否生效
