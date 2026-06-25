# fs-aoi-web 项目技术栈与开发规范

> 项目名称：kjdp-webapp (fs-aoi-web)  
> 版本：3.10.0  
> 定位：证券业务综合服务平台 - 前端门户应用

---

## 一、技术栈总览

### 1.1 核心框架

| 技术           | 版本   | 用途                        |
| -------------- | ------ | --------------------------- |
| **Vue**        | 3.4.15 | 前端框架（Composition API） |
| **Vue Router** | 4.2.5  | 路由管理（Hash 模式）       |
| **Pinia**      | 2.1.7  | 状态管理                    |
| **Vite**       | 5.0.8  | 构建工具 + 开发服务器       |

### 1.2 公司私有库

| 包名                        | 版本   | 用途                                                   |
| --------------------------- | ------ | ------------------------------------------------------ |
| `@szkingdom.kjdp/core`      | 1.15.2 | 平台核心库（HTTP 请求、加密、系统工具、会话管理）      |
| `@szkingdom.kjdp/ui`        | 2.3.60 | UI 组件库（基于 Element Plus 封装，含 Kui\* 系列组件） |
| `@szkingdom.kjdp/icons-vue` | 1.0.2  | 图标字体库                                             |

### 1.3 生态依赖

| 包名                        | 版本           | 用途                                 |
| --------------------------- | -------------- | ------------------------------------ |
| `echarts`                   | 6.0.0          | 数据可视化图表                       |
| `xgplayer`                  | 3.0.23         | 视频播放器                           |
| `docxtemplater` + `pizzip`  | 3.67.5 / 3.2.0 | Word 文档模板生成                    |
| `xlsx`                      | 0.18.5         | Excel 解析与导出                     |
| `markdown-it`               | 14.1.0         | Markdown 渲染                        |
| `markdown-it-multimd-table` | 4.2.3          | Markdown 多格式表格扩展              |
| `highlight.js`              | 11.11.1        | 代码语法高亮                         |
| `@vueuse/core`              | 10.9.0         | Vue 组合式工具函数                   |
| `@vueuse/components`        | 10.9.0         | VueUse 组件（如拖拽等）              |
| `vue-draggable-plus`        | 0.6.0          | 拖拽排序                             |
| `lodash-es`                 | 4.17.21        | 工具函数库（ESM 版本）               |
| `mitt`                      | 3.0.1          | 事件总线                             |
| `uuid`                      | 9.0.1          | UUID 生成                            |
| `angular-expressions`       | 1.5.1          | 模板表达式引擎（配合 docxtemplater） |
| `jsonp`                     | 0.2.1          | JSONP 请求                           |

### 1.4 开发依赖

| 包名                      | 版本   | 用途                      |
| ------------------------- | ------ | ------------------------- |
| `@vitejs/plugin-vue`      | 4.5.2  | Vite Vue 插件             |
| `sass`                    | 1.69.5 | SCSS 预处理器             |
| `eslint`                  | 8.55.0 | 代码检查                  |
| `prettier`                | 3.1.1  | 代码格式化                |
| `eslint-plugin-vue`       | 9.19.2 | Vue ESLint 规则           |
| `eslint-config-prettier`  | 9.1.0  | Prettier 兼容             |
| `eslint-plugin-prettier`  | 5.0.1  | Prettier 作为 ESLint 规则 |
| `vite-plugin-compression` | 0.5.1  | 构建压缩                  |
| `rollup-obfuscator`       | 3.0.2  | 代码混淆                  |
| `cross-env`               | 10.1.0 | 跨平台环境变量            |
| `fast-glob`               | 3.3.3  | 快速文件匹配              |

### 1.5 自定义 Vite 插件

| 插件名                      | 文件                                        | 用途                                                                         |
| --------------------------- | ------------------------------------------- | ---------------------------------------------------------------------------- |
| `local-dir-proxy`           | `config/plugins/local-dir--proxy/`          | 开发时拦截 `*web` 路径请求，代理到本地文件系统目录                           |
| `versioned-resource-loader` | `config/plugins/versioned-resource-loader/` | 生产构建时为所有 JS/CSS 资源添加版本号参数，并注入运行时脚本拦截动态资源加载 |

---

## 二、项目结构

```
fs-aoi-web/
├── config/                    # 构建配置
│   ├── build.js               # Rollup 构建选项（chunk 分割、hash 模式）
│   ├── dev.js                 # 开发服务器配置（端口、代理）
│   ├── plugin.js              # Vite 插件注册
│   └── plugins/               # 自定义 Vite 插件
├── public/static/             # 静态资源（不参与构建）
├── src/
│   ├── main.js                # 应用入口
│   ├── App.vue                # 根组件（KeepAlive + RouterView）
│   ├── assets/styles/         # 全局样式
│   │   ├── var.scss           # CSS 变量（主题 token）
│   │   ├── global.scss        # 全局样式
│   │   ├── index.scss         # 样式入口
│   │   └── kjdp-ui/           # UI 库样式覆盖
│   ├── config/                # 应用配置
│   │   ├── index.js           # 配置聚合导出
│   │   ├── http.js            # HTTP / API 配置
│   │   ├── services.js        # 服务接口号配置
│   │   ├── service-code-map.js # 接口号到服务名的映射
│   │   ├── webapp.js          # 门户 / 菜单 / 项目配置
│   │   ├── kjdp.js            # UI 组件全局默认属性
│   │   ├── callbacks.js       # 生命周期回调
│   │   ├── consts.js          # 业务常量
│   │   └── kone-adapter.js    # KOne 适配
│   ├── pages/                 # 业务页面模块（按子系统划分）
│   │   ├── aoi/               # AOI 主业务（流程受理/审核、搜索驾驶舱等）
│   │   ├── cop/               # COP 协办业务（客户认证、身份核查等）
│   │   ├── frame/             # 框架层（工作台视图）
│   │   ├── idm/               # IDM 影像管理（摄像头/视频/PDF/签名）
│   │   ├── isc/               # ISC 智能服务（AI 相关）
│   │   └── uas/               # UAS 账户服务（客户推荐、通用组件）
│   ├── portal/                # 门户框架
│   │   ├── index.js           # 门户入口（初始化链路）
│   │   ├── hooks/             # 组合函数（主题、菜单、消息、SSO等）
│   │   ├── modules/           # 功能模块（tabs、highlight、通用审核等）
│   │   ├── router/            # 路由配置（动态路由组装）
│   │   ├── store/             # Pinia 全局状态
│   │   └── views/             # 视图（login、layout、workbench）
│   └── themes/                # 主题配置（dark/deepblue/default/fs25/light/yellow）
├── .eslintrc.js               # ESLint 配置
├── .prettierrc                # Prettier 配置
├── .npmrc                     # 包管理器配置
├── jsconfig.json              # 路径别名
├── vite.config.js             # Vite 主配置
└── package.json
```

---

## 三、路径别名

| 别名      | 实际路径                                          | 使用场景     |
| --------- | ------------------------------------------------- | ------------ |
| `@`       | `./src`                                           | 通用源码引用 |
| `@assets` | `./src/assets`                                    | 样式/资源    |
| `@config` | `./src/config`                                    | 配置文件     |
| `@pages`  | `./src/pages`                                     | 业务页面     |
| `@portal` | `./src/portal`                                    | 门户框架     |
| `@hooks`  | `./src/portal/hooks`                              | 组合函数     |
| `@static` | 动态：开发为 `./static`，生产为 `./public/static` | 静态资源     |

---

## 四、ESLint 代码规范

**解析器：** `vue-eslint-parser`（支持 Vue 3 + JSX）  
**规则集继承：** `plugin:vue/vue3-recommended` + `plugin:prettier/recommended`

### 核心规则

| 规则                             | 配置                     | 说明                                   |
| -------------------------------- | ------------------------ | -------------------------------------- |
| `no-var`                         | `error`                  | **禁止使用 var**，必须用 `let`/`const` |
| `prefer-const`                   | `error`                  | **优先使用 const**                     |
| `no-unused-vars`                 | `error`                  | **禁止未使用变量**                     |
| `vue/multi-word-component-names` | `off`                    | 允许单词组件名                         |
| `no-extend-native`               | `off`                    | 允许扩展原生对象                       |
| `no-eval`                        | `off`                    | 允许间接使用 eval                      |
| `no-console`                     | 生产 `warn` / 开发 `off` |                                        |
| `no-debugger`                    | 生产 `warn` / 开发 `off` |                                        |

### 忽略范围

- `./public` 目录不参与 lint

---

## 五、Prettier 格式化规范

```json
{
  "useTabs": false, // 使用空格缩进
  "tabWidth": 2, // 2 空格缩进
  "printWidth": 120, // 行宽 120 字符
  "singleQuote": true, // 单引号
  "trailingComma": "none", // 无尾逗号
  "bracketSpacing": true, // 对象花括号空格 { a: 1 }
  "semi": false, // 无分号
  "arrowParens": "avoid", // 箭头函数单参数省略括号 x => x
  "endOfLine": "auto" // 自动换行符
}
```

---

## 六、CSS / SCSS 规范

### 6.1 预处理器

- 使用 **SCSS**（`sass 1.69.5`）
- 编译器：`modern-compiler` API
- 全局注入：`@use "@assets/styles/var.scss" as *`（所有 SCSS 文件自动可用）

### 6.2 主题变量体系

- 通过 **CSS 自定义属性（CSS Variables）** 实现主题切换
- 变量定义在 `src/assets/styles/var.scss`
- 前缀规范：`--fs-*`
- 主题文件位于 `src/themes/`，包含：`dark`、`deepblue`、`default`、`fs25`（默认）、`light`、`yellow`

### 6.3 变量分类

| 类别        | 前缀示例                     | 说明                     |
| ----------- | ---------------------------- | ------------------------ |
| 色彩        | `--fs-color-primary`         | 主色/成功/警告/危险/信息 |
| 字体        | `--fs-font-size-base`        | 字体大小（min~max 7 级） |
| 字体颜色    | `--fs-font-color-base`       | 基础/对比字体色          |
| 背景        | `--fs-background-color-base` | 基础/对比/选中背景色     |
| 边框        | `--fs-border-color-base`     | 边框颜色                 |
| 圆角        | `--fs-border-radius-base`    | 圆角大小                 |
| 阴影        | `--fs-box-shadow-base`       | 阴影样式                 |
| 门户 Header | `--fs-portal-header-*`       | 门户导航栏相关           |
| 左侧菜单    | `--fs-left-menu-*`           | 菜单面板相关             |
| 内容 Tabs   | `--fs-content-tabs-*`        | 标签页相关               |
| 内容主区    | `--fs-content-main-*`        | 主内容区域               |
| Dialog      | `--fs-dialog-header-*`       | 弹窗样式                 |

---

## 七、HTTP 请求规范

### 7.1 请求架构

- 基于 `@szkingdom.kjdp/core` 的 `request` 模块
- 支持 `request.service(serviceCode, data, config)` 调用后端 FS 服务
- 支持 `request.request(url, method, data, config)` 调用 REST 接口

### 7.2 服务路由映射

- 接口号前缀决定目标服务：

| 前缀     | 服务       | 路径                    |
| -------- | ---------- | ----------------------- |
| `P00`    | 平台通用   | `/api/fs-aoi/fsapi`     |
| `F01`    | UAS 账户   | `/api/fs-uas-mng/fsapi` |
| `F5901`  | IDM 影像   | `/api/fs-idm/fsapi`     |
| `F5902`  | ISC 智能   | `/api/fs-isc/fsapi`     |
| `F09`    | AOI 主业务 | `/api/fs-aoi/fsapi`     |
| `D` 开头 | IDM 影像   | `/api/fs-idm/fsapi`     |
| `I` 开头 | ISC 智能   | `/api/fs-isc/fsapi`     |

### 7.3 错误处理

- 成功码：`MSG_CODE` 为 `'0'` 或 `'100'`
- 错误时弹出 `KuiMessageBox.alert` 显示详细错误信息（含 traceId）
- 支持加密传输（`fsEncrypt: true`）

### 7.4 开发代理

| 路径      | 目标                      | 说明             |
| --------- | ------------------------- | ---------------- |
| `/copweb` | 本地文件系统路径          | COP 前端静态资源 |
| `/uasweb` | 本地文件系统路径          | UAS 前端静态资源 |
| `/idmweb` | 本地文件系统路径          | IDM 前端静态资源 |
| `/api`    | `http://10.201.69.2:8080` | API 网关代理     |

---

## 八、路由规范

### 8.1 路由模式

- **Hash 模式**（`createWebHashHistory`）
- 支持 URL 参数加密（`projectConfig.urlEncrypt`）

### 8.2 路由结构

| 路径                   | 名称         | 组件                             | 说明         |
| ---------------------- | ------------ | -------------------------------- | ------------ |
| `/`                    | —            | 重定向 `/login`                  | 根路由       |
| `/login`               | `login`      | `@portal/views/login/index.vue`  | 登录页       |
| `/portal`              | `portal`     | `@portal/views/layout/index.vue` | 门户主布局   |
| `/portal/:portalKey/*` | 动态         | 动态加载                         | 业务菜单页面 |
| `/signScreen`          | `signScreen` | IDM 签名屏                       | 独立页面     |
| `/collect`             | `collect`    | IDM 采集页                       | 独立页面     |

### 8.3 路由守卫逻辑

1. iframe 内嵌模式：路径变化时向父窗口发消息阻止路由跳转
2. 未登录访问非 login 页面 → 重定向到 `/login`
3. 已登录访问 login → 重定向到 `/portal`
4. 直接访问非 portal 的路由 → 重定向到 `/portal`

### 8.4 动态路由

- 通过 `import.meta.glob('@pages/*/index.js')` 自动扫描页面模块
- 各模块在 `index.js` 中导出 `routes` 配置
- 框架自动组装到 `/portal` 子路由下

---

## 九、状态管理规范

### 9.1 Store 结构

- 使用 **Pinia** 的 `defineStore` + Options API 风格
- 全局 Store：`usePortalStore`（`src/portal/store/index.js`）

### 9.2 核心状态

| 状态               | 类型    | 用途               |
| ------------------ | ------- | ------------------ |
| `opInfo`           | Object  | 当前登录员工信息   |
| `portalData`       | Array   | 门户配置数据       |
| `menuTree`         | Array   | 菜单树             |
| `menuArr`          | Array   | 菜单扁平列表       |
| `openedTabs`       | Object  | 已打开标签页       |
| `openedIframeTabs` | Array   | 已打开 iframe 标签 |
| `isMenuCollapse`   | Boolean | 菜单折叠状态       |

---

## 十、UI 组件全局配置

> 配置文件：`src/config/kjdp.js`

| 组件             | 默认属性                                             |
| ---------------- | ---------------------------------------------------- |
| `KuiInput`       | `clearable: true`                                    |
| `KuiDialog`      | `closeOnClickModal: false, closeOnPressEscape: true` |
| `KuiDrawer`      | `closeOnClickModal: true, closeOnPressEscape: true`  |
| `KuiMessageBox`  | `showClose: false`                                   |
| `KuiTable`       | `size: 'default', stripe: true, border: false`       |
| `KuiForm`        | `clearable: true`                                    |
| `KuiGeneralForm` | `errorRender: 'text'`                                |
| `KuiField`       | `isBlock: true`                                      |
| `KuiComboTree`   | `checkStrictly: true`                                |
| `TableButtons`   | `align: 'left', inline: true`                        |
| `TableDialog`    | `dialogWidth: 700`                                   |

---

## 十一、构建规范

### 11.1 构建模式

| 模式                 | 触发方式                                           | 说明                 |
| -------------------- | -------------------------------------------------- | -------------------- |
| **版本模式**（默认） | `cross-env APP_VERSION="x.x.x.x(R)" npm run build` | 资源路径携带版本参数 |
| **Hash 模式**        | `cross-env BUILD_MODE=hash npm run build`          | 文件名使用内容 hash  |

### 11.2 Chunk 分割策略

- **同步依赖**：合并到 `dependence/chunks`
- **异步依赖**：按包名独立拆分（echarts、highlight.js、xgplayer、xlsx、markdown-it、docxtemplater、pizzip）
- **业务代码**：`src/pages/` 下保留原始目录结构
- **框架代码**：合并到 `src/frame`

### 11.3 生产优化

- `esbuild.drop: ['console', 'debugger']`：移除控制台日志
- `sourcemap: true`：生成 source map
- 支持代码混淆（`rollup-obfuscator`）
- 支持 gzip 压缩（`vite-plugin-compression`）

---

## 十二、包管理规范

### 12.1 包管理器

- 支持 **pnpm**（推荐）和 **npm**
- Node 版本要求：`^18.0.0 || ^20.0.0 || >=22.0.0`

### 12.2 npmrc 配置

```ini
shamefully-hoist = true          # 提升所有依赖到顶层（兼容非 pnpm 友好包）
strict-peer-dependencies = false  # 不严格检查 peer 依赖
auto-install-peers = true         # 自动安装 peer 依赖
legacy-peer-deps = true           # 允许 npm 旧式 peer 依赖解析
```

### 12.3 NPM Scripts

| 命令            | 说明                        |
| --------------- | --------------------------- |
| `pnpm dev`      | 启动开发服务器（端口 8080） |
| `pnpm build`    | 生产构建                    |
| `pnpm preview`  | 预览构建产物                |
| `pnpm lint`     | 代码检查                    |
| `pnpm lint:fix` | 自动修复代码问题            |
| `pnpm format`   | 代码格式化                  |

---

## 十三、应用生命周期

```
main.js
  ├── createApp(App)
  ├── app.use(createPinia())          → Pinia 状态管理
  ├── app.use(KjdpCore, config)       → 核心库初始化（HTTP、加密、会话等）
  ├── app.use(KjdpUI, config)         → UI 组件库注册
  ├── callbackMap.appBeforeMount()     → 挂载前初始化
  │   ├── getUrlEncryptKey()           → URL 加密密钥
  │   ├── iframe 模式相关初始化
  │   ├── checkIsSubSysMode()          → 子系统模式检查
  │   └── callbacks.appBeforeMount()   → 系统参数缓存加载
  ├── app.use(router)                  → 路由注册
  ├── app.mount('#app')               → DOM 挂载
  └── callbackMap.appMounted()         → 挂载后初始化
      ├── useFavicon.init()            → 网站图标
      ├── useThemes.init()             → 主题初始化
      └── callbacks.appMounted()       → 版本检查（每 8 小时轮询）
```

---

## 十四、业务模块职责

| 模块      | 目录          | 核心职责                                                    |
| --------- | ------------- | ----------------------------------------------------------- |
| **AOI**   | `pages/aoi`   | 业务受理/审核、流程管理、搜索驾驶舱、数据项管理、适当性管理 |
| **COP**   | `pages/cop`   | 客户身份认证、CSDC 核查、密码验证、三方信息查询             |
| **Frame** | `pages/frame` | 工作台视图、框架扩展                                        |
| **IDM**   | `pages/idm`   | 影像采集（摄像头/视频/PDF/签名）、多媒体处理                |
| **ISC**   | `pages/isc`   | AI 智能服务、电子表单配置                                   |
| **UAS**   | `pages/uas`   | 账户管理、客户推荐、城市选择、通用文件上传                  |

---

## 十五、编码约定总结

1. **变量声明**：只用 `const`（优先）和 `let`，禁止 `var`
2. **引号**：单引号
3. **分号**：不加分号
4. **缩进**：2 空格
5. **行宽**：120 字符
6. **尾逗号**：不加
7. **箭头函数**：单参数省略括号
8. **组件命名**：允许单词组件名
9. **未使用变量**：禁止（ESLint error）
10. **样式方案**：SCSS + CSS Variables 主题体系
11. **状态管理**：Pinia Options API 风格
12. **路由模式**：Hash 模式
13. **HTTP 请求**：通过 `request.service(serviceCode)` 调用，接口号自动路由到对应后端服务
