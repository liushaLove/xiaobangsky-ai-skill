# copweb 项目技术栈与开发规范

> 项目名称：copweb / KCOP
> 项目定位：一柜通 / 综合运营平台前端静态资源工程  
> 适用范围：`F:\git\kcop-zh-project\aoi\20250721\copweb`

本仓库不是 Vue/Vite/Pinia 项目，而是 KJDP4/KUI 体系下的 legacy 多子系统前端。维护时应优先遵循现有 AMD 模块、jQuery/KUI 组件、Backbone 视图、Grunt 构建与 tmod 模板编译方式。

---

## 一、技术栈总览

| 类型 | 技术 / 库 | 项目中的角色 |
| --- | --- | --- |
| 模块系统 | `loader.js` + `define(function(require, exports, module){ ... })` | 页面与公共模块加载，语义接近 SeaJS/CMD |
| 基础框架 | KJDP4.0 / KUI 4.0.3 | 门户、组件、请求、缓存、菜单、字典、机构等基础能力 |
| DOM / 异步 | jQuery + Deferred | DOM 操作、组件调用、`$.when`/`Deferred` 异步流 |
| 视图组织 | Backbone.Model / Backbone.View | 业务页常用模型与视图封装 |
| 工具库 | Underscore / lodash 风格 `_` | 集合处理、对象合并、过滤、查找 |
| 模板 | artTemplate / tmod | HTML 模板预编译到 `template.js` |
| UI 组件 | KUI jQuery 插件 | `textinput`、`combobox`、`combotree`、`datagrid`、`form`、`dialog`、`obviousbox` 等 |
| 样式 | CSS / LESS / SCSS | 业务 CSS、KUI 主题、Bootstrap LESS、`frame/themes` SCSS |
| 构建 | Grunt | 复制、压缩、CMD transport/concat、uglify、模板编译、版本指纹 |

重要判断：不要引入 Vue、Vite、Pinia、ESM 或现代打包范式，除非用户明确要求大规模迁移。

---

## 二、关键入口与加载链路

### 2.1 顶层入口

- 根 `index.html` 通过脚本跳转到 `apps/opp/index.html`。
- 业务页面一般在 HTML 中引入根目录 `loader.js`，并通过 `data-main` 指定主模块。
- 典型页面入口：

```html
<script id="seajsnode"
        type="text/javascript"
        src="../../../../loader.js"
        data-iskui="true"
        data-main="./src/reset-cust-org"></script>
```

### 2.2 加载器配置

- `loader-config.js`
  - `proName: "KCOP"`
  - `proVersion: "1.0.2.0"`
  - `frameReady: "basic/ready-kdop"`
  - `topWinPages` 配置允许直接打开的顶层页面
  - `bizReady` 预留子系统 ready 模块
- `loader-frame-config.js`
  - 框架名 `kui`
  - 框架版本 `4.0.3`
  - 框架核心加载顺序：`frame/lib` -> `frame/core` -> `frame/kui`
  - 主题：`smart-blue`、`deep-blue`、`blue`
  - `adaptKdop: true`、`isKdop: true`

### 2.3 基础初始化

- `basic/ready.js` 是业务基础初始化核心。
- 初始化内容包括：
  - 挂载/复用顶层 `kcopTop.kui`
  - 加载 `basic/modules/*` 工具、字典、机构、参数、菜单模型
  - 加载 `basic/service/*` 服务封装
  - 加载登录后缓存：系统、节点、机构、省市区、复核配置、KIDM 地址等
  - 对 KUI 插件做业务默认值覆盖，如 datagrid 权限、字典联动、localstorage 前缀

---

## 三、目录结构与职责

```text
copweb/
├─ apps/                  # 主要业务页面，opp 是核心子应用
│  ├─ opp/
│  │  ├─ index.html        # 综合运营入口
│  │  ├─ pages/            # 入口页、菜单、首页等
│  │  ├─ operator/         # 柜员/办理类页面
│  │  ├─ manager/          # 管理配置类页面
│  │  ├─ review/           # 复核/审核类页面
│  │  ├─ settlement/       # 清算/结算类页面
│  │  ├─ common/           # opp 公共模块
│  │  └─ frame/            # opp 门户框架资源
│  └─ tpls/                # tmod 业务模板源与编译产物
├─ basic/                  # 基础业务层：ready、公共 service/module/view
├─ frame/                  # KJDP/KUI 框架层：core、kui、views、themes
├─ lib/                    # 第三方库与 KUI/web2 组件库
├─ upload/                 # Excel/DBF 模板、业务视频图片等静态资源
├─ kcop/ kuas/ kidm/ ...   # 多子系统静态资源或兼容页面
├─ loader.js               # 项目加载器
├─ loader-config.js        # 项目级加载配置
├─ loader-frame-config.js  # 框架级加载配置
└─ Gruntfile.js            # 构建与模板编译任务
```

常见子系统目录：`kcop`、`kdrf`、`kfis`、`kfms`、`kgis`、`kgob`、`kidh`、`kidm`、`kisc`、`kmos`、`kone`、`kscs`、`kspb`、`kuas`、`cicc`。

---

## 四、模块与页面编写规范

### 4.1 AMD/CMD 模块格式

业务 JS 统一使用：

```javascript
define(function (require, exports, module) {
    var someService = require("../../../common/service/some-service");

    exports.$ready = function () {
        // 页面初始化
    };
});
```

约定：

- 主页面模块通常导出 `exports.$ready`，由 loader 在页面加载完成后调用。
- 依赖路径优先使用仓库既有相对路径或以 `apps/...`、`basic/...`、`frame/...` 开头的模块路径。
- 不要使用 `import/export`、`async/await`、TypeScript、Vue SFC。
- 兼容老浏览器语法，优先使用 `var`、普通函数、字符串拼接；避免未转译的新语法。

### 4.2 页面 HTML 规范

业务页面通常由三类文件组成：

```text
some-page.html
src/some-page.js
css/some-page.css
```

HTML 负责：

- 引入页面 CSS。
- 引入 `loader.js` 并配置 `data-main`。
- 编写 KUI 表单/组件占位 DOM。
- 通过 `class="kui-xxx"` 与 `kui-options="..."` 声明 KUI 组件。

示例组件声明：

```html
<input id="ORG_CODE"
       name="ORG_CODE"
       class="kui-combotree"
       kui-options="width:190,required:'true',req:[{service:'P000000200'}]" />
```

注意：

- `kui-options` 是字符串形式的组件配置，不要随意改成 JSON。
- 与 JS 中通过 `$("#ORG_CODE").combotree(...)`、`form("load")`、`form("disable")` 等调用配套维护。
- 修改字段 `id/name` 时，必须同步检查 JS 选择器、表单 load/save 字段和后端字段名。

### 4.3 Backbone 使用习惯

典型业务页会拆成：

- `Backbone.Model.extend({ defaults, getXxx })`：封装 ajax/service 数据获取。
- `Backbone.View.extend({ initialize, validate, getData, setData })`：封装 DOM、组件和业务交互。
- 页面级 `exports.$ready`：解析 URL、加载数据、初始化 flow 或主视图。

维护时优先沿用当前页面结构，不要为单点修改引入新框架或复杂抽象。

---

## 五、请求与后端服务规范

### 5.1 ajax 基础

核心模块：`frame/core/ajax.js`。

常用调用：

```javascript
ajax.post({
    req: {
        service: "P9999999",
        bex_codes: "YGT_A1100807",
        CUST_CODE: custCode
    }
}).then(function (data, head, answers) {
    // data 通常是二维数组结构
});
```

约定：

- `ajax.post` 实际走 `ajax.ajax4kui`。
- `req.service` 是必填服务号。
- 后端结果常见结构为 `data[0][0]` 或 `data[0]`，取值前要保留空值保护。
- 成功码由 `ajax.isSuccess` 判断，默认包括 `"0"`、`"100"`、`"9010001"`、`"-130011"`、`"-404"`、`"106192"`，也可通过 `extMsgCode` 扩展。
- 默认错误会统一弹窗；特殊场景可用 `noProcess`、`showErrorPrompt: false` 控制。

### 5.2 合并请求

老 KJDP 模式支持：

```javascript
ajax.start();
var req1 = serviceA();
var req2 = serviceB();
ajax.send();
```

注意：

- `_fs` 模式下合并请求会被关闭或拆分。
- 不要随意改变 `ajax.start()`/`ajax.send()` 包围范围，可能影响已有请求顺序与回调结构。

### 5.3 公共服务层

公共服务多在：

- `basic/service/*`
- `apps/opp/common/service/*`

优先复用已有服务方法，如 `cust-service`、`csdc-service`、`flow-service`、`kisc-service`、`lbm-service`、`menu-service`。只有没有合适封装时才在页面内直接写 `ajax.post`。

---

## 六、KUI 组件使用规范

常见组件与调用：

| 组件 | HTML class / JS 调用 | 说明 |
| --- | --- | --- |
| 文本框 | `kui-textinput` / `.textinput()` | 输入与校验 |
| 密码框 | `kui-password` / `.password()` | 证件号、密码等敏感字段 |
| 下拉框 | `kui-combobox` / `.combobox()` | 字典、固定选项 |
| 下拉树 | `kui-combotree` / `.combotree()` | 机构、菜单、层级数据 |
| 表单 | `kui-form` / `.form("load")` | 数据加载、禁用、校验 |
| 表格 | `.datagrid()` | 查询列表、工具栏、权限按钮 |
| 弹窗 | `.dialog()` / `kcopTop.alert` | 业务提示与弹窗 |
| obviousbox | `.obviousbox()` | 单选/多选视觉控件 |

维护原则：

- 初始化组件后再调用组件方法。
- 组件值读取要使用组件 API，例如 `combotree("getValue")`、`combotree("getValues")`、`obviousbox("getValue")`。
- 表单校验优先调用组件自身 `isValid` 或 `form("validate")`。
- 页面提示优先使用当前模块已有方式，如 `flow.showFlowTip`、`kcopTop.alert`、`loading(...)`。

---

## 七、模板规范

### 7.1 tmod 模板

- 源模板：`apps/tpls/**/*.html`
- 编译产物：`apps/tpls/src/template.js`
- helper：`apps/tpls/helper.js`
- Grunt 任务：`grunt tmod:apps`
- 快捷脚本：`build-tpls.bat`

修改 `apps/tpls/**/*.html` 后必须重新编译模板。

### 7.2 页面内 artTemplate

部分页面直接使用：

```javascript
var render = artTemplate("loginWrapTpl");
```

此类模板通常来自页面 HTML 中的 `<script type="text/html" id="...">`，修改模板 id 时必须同步 JS。

---

## 八、样式与主题规范

| 位置 | 用途 |
| --- | --- |
| `apps/**/css/*.css` | 业务页面局部样式 |
| `frame/themes/*.css` | 框架主题入口 |
| `frame/themes/smart-blue/**/*.scss` | smart-blue 主题源样式 |
| `lib/kui/themes/*.css` | KUI 组件主题 |
| `lib/bootstrap/css/bootstrap.less` | Bootstrap/KIF 样式源 |

规范：

- 页面级样式优先放在页面附近 `css/` 目录。
- 不要随意改 `lib/`、`frame/themes/` 全局样式，影响面很大。
- 业务页新增样式应通过页面 HTML 显式 `<link>` 引入。
- 旧页面大量使用固定宽度、浮动和行内布局，修改布局时要检查既有 KUI 组件尺寸和弹层宽高。

---

## 九、构建与常用命令

本仓库根目录没有标准现代 `package.json` 脚本。构建入口以 Grunt 和 `.bat` 为主。

### 9.1 Grunt 默认构建

`Gruntfile.js` 默认任务主要执行：

1. `clean:build`
2. `copy:web`
3. `copy:server`
4. `cssmin`
5. `transport:kjdp`
6. `transport:index`
7. `concat:kjdp`
8. `concat:index`
9. `uglify:kjdp`
10. `uglify:index`
11. `replace:index`
12. `copy:build`
13. `copy:report`

输出目录：

- 临时/前端产物：`dist/`
- 汇总发布目录：`../../kcop-dist/`

### 9.2 模板编译

```bat
build-tpls.bat
```

等价核心步骤：

```bat
grunt clean:appsTpls
grunt tmod:apps
```

### 9.3 其他脚本

| 脚本 | 用途 |
| --- | --- |
| `build.bat` | 触发默认构建 |
| `build-all.bat` | 全量构建/拷贝类脚本 |
| `build-kui-css.bat` | KUI CSS 相关构建 |
| `build-db-doc.bat` | 数据库文档相关构建 |
| `build-kwas.bat`、`build-tongweb.bat` | 特定发布形态脚本 |

执行构建前注意：此项目依赖老版本 Node/Grunt 环境和已有 `node_modules`，不要默认升级依赖。

---

## 十、代码风格与兼容性

此仓库无根级 ESLint/Prettier 配置。风格以历史代码为准：

- 缩进多为 4 空格。
- 字符串多使用双引号，也有单引号混用；修改局部时保持周边一致。
- 大量代码使用 `var`，不要机械改成 `let/const`。
- 大量异步使用 jQuery Deferred：`.then`、`.done`、`.fail`、`.always`。
- 不要引入 Babel/webpack/Vite 转译假设。
- 避免使用可破坏老浏览器兼容性的语法，如可选链、空值合并、class fields、顶层 await。
- 新增全局变量前先检查 `kcopTop`、`kui`、`consts`、`utils`、`ajax`、`cache` 是否已有约定。

---

## 十一、常见业务模式

### 11.1 flow 节点页

办理流程页常见：

```javascript
flow.init({
    text: "资料录入",
    el: "#infoNodeWrap",
    init: function ($el, busiData, nodeData) {},
    validate: function () {},
    save: function () {}
});
```

维护要点：

- `init` 负责加载数据和创建视图。
- `validate` 返回布尔值，失败时通常调用 `flow.showFlowTip`。
- `save` 返回写回流程上下文的数据对象。
- 不要改变 `busiData` 字段结构，除非同步确认后续节点和保存接口。

### 11.2 菜单/门户页

入口页如：

- `apps/opp/pages/src/index.js`
- `apps/opp/operator/pages/src/index.js`
- `apps/opp/manager/pages/src/index.js`
- `apps/opp/review/pages/src/index.js`
- `apps/opp/settlement/pages/src/index.js`

这些文件被 `Gruntfile.js` 的 `indexJsArr` 固定列入 transport/concat/uglify。修改入口依赖时要注意构建任务中的依赖解析。

### 11.3 静态模板与导入文件

- Excel 模板位于 `upload/excel-tpl/` 或业务目录 `file/`。
- DBF 模板位于 `upload/dbf-tpl/`。
- 文件名常与业务代码、导入导出逻辑、后端配置强绑定，重命名需谨慎。

---

## 十二、修改前定位建议

1. 从用户给出的页面名、菜单名、业务码、服务号、字段名入手，用 `rg -n` 搜索。
2. 找到 HTML 后查看 `data-main`，再进入对应 `src/*.js`。
3. 找 JS 中的 `exports.$ready`、`flow.init`、`Backbone.View.initialize`、`validate`、`save/getData`。
4. 查服务调用：`ajax.post({ req: { service, bex_codes } })` 或 `basic/service`、`apps/opp/common/service`。
5. 查字段链路：HTML `id/name` -> JS `$el.find` -> `form("load")` -> `getData/save` -> ajax 参数。
6. 如果涉及模板，检查 `apps/tpls/**/*.html` 与 `apps/tpls/src/template.js`。

---

## 十三、维护红线

- 不要把本仓库当成 Vue/Vite 项目处理。
- 不要删除或重命名 `loader.js`、`loader-config.js`、`loader-frame-config.js`。
- 不要随意改 `frame/core/ajax.js`、`basic/ready.js`、`frame/kui.js`、`lib/kui/**` 等全局基础层。
- 不要格式化大文件或批量现代化语法，容易造成不可控差异。
- 不要在没有验证的情况下升级 `node_modules`、Grunt 插件或 KUI 组件库。
- 不要只改 HTML 字段而不查 JS、服务参数、模板和 CSS。
- 修改模板源后不要忘记重新生成 `apps/tpls/src/template.js`。

---

## 十四、推荐验证方式

根据改动类型选择最小验证：

| 改动类型 | 推荐验证 |
| --- | --- |
| 单个业务 JS | 检查 `define`、括号、依赖路径；必要时用页面手工打开验证 |
| HTML/KUI 字段 | 同步检查 JS 选择器、form load/save、组件 API |
| CSS | 检查页面静态渲染和 KUI 组件尺寸 |
| tmod 模板 | 运行 `build-tpls.bat` 或 `grunt tmod:apps` |
| 入口页/公共模块 | 运行相关 Grunt 构建任务，并扩大页面回归范围 |
| ajax/service | 验证成功、失败、会话超时和空数据分支 |

如果只能做静态验证，应在交付说明里明确“未运行构建/未打开页面”的限制。
