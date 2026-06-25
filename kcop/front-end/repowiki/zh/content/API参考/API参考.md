# API 参考

当前项目的 API 不是 Vue composable 或 npm 包导出，而是 KJDP4/KUI legacy 全局对象与 AMD 模块。

## API 分类

- 请求：`ajax.post`、`ajax.get`、`ajax.start`、`ajax.send`、`ajax.socket`
- 组件：KUI jQuery 插件，如 `form`、`combobox`、`combotree`、`datagrid`、`dialog`、`obviousbox`
- 工具：`utils`、`consts`、`cache`
- 框架：`kui`、`kcopTop.kui`
- 服务：`basic/service/*`、`apps/opp/common/service/*`
- 视图：`Backbone.Model`、`Backbone.View`

优先阅读同目录下 `服务接口.md`、`组件API.md`、`工具函数.md`。
