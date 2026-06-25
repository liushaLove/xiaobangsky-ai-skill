# Hook 接口

当前 copweb 没有 Vue composable/hook 体系。旧文档中的 `useXxx` 不适用。

| Vue Hook 概念 | copweb 对应方式 |
| --- | --- |
| `useRouter` | 菜单 URL、iframe/标签页、`utils.buildUrl` |
| `useStore` | `kcopTop.kui`、`kui.basic`、缓存、Backbone Model |
| `useRequest` | `ajax.post` 或 service 模块 |
| 生命周期 hook | `exports.$ready`、`Backbone.View.initialize`、`flow.init` |
