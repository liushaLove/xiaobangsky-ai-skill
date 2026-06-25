# KJDP/KUI 核心 API 参考

> 当前 copweb 不使用 `@szkingdom.kjdp/core` npm 包。本页描述的是本项目实际存在的 KJDP4/KUI legacy API：全局 `ajax`、`kui`、`utils`、KUI jQuery 插件和公共 service。

## 全局对象

| 对象 | 来源 | 用途 |
| --- | --- | --- |
| `ajax` | `frame/core/ajax.js` | KJDP 后端请求封装 |
| `kui` | `frame/core.js`、`frame/kui.js`、`basic/ready.js` | KUI 框架、组件、缓存、字典、机构等能力 |
| `kcopTop` | 顶层窗口约定 | 跨 iframe/页面共享状态和弹窗 |
| `utils` | 框架工具 | URL 参数、加密、JSON、页面上下文等 |
| `consts` | 基础常量 | 服务 URL、登录 token、菜单常量等 |
| `cache` | 框架缓存 | 模块和业务缓存 |

## ajax.post

```javascript
ajax.post({
    req: {
        service: "P9999999",
        bex_codes: "YGT_A1100807",
        CUST_CODE: custCode
    }
}).then(function (data, head, answers, jqXHR) {
    var row = data && data[0] && data[0][0] || {};
});
```

规则：`req.service` 必填；返回值是 jQuery Deferred；数据常见形态是二维数组；错误默认由框架弹窗处理。

## ajax.start / ajax.send

```javascript
ajax.start();
var a = serviceA();
var b = serviceB();
ajax.send();
```

老 KJDP 模式下用于合并初始化请求。`kcopTop._fs` 模式下合并逻辑会被关闭或拆分。

## 成功码

`ajax.isSuccess(head, extMsgCode)` 默认认可 `0`、`100`、`9010001`、`-130011`、`-404`、`106192`。

## KUI 组件 API

```javascript
$form.form("load", data);
$form.form("disable", ["CUST_CODE"]);
$org.combotree("getValue");
$org.combotree("setValue", orgCode);
$flag.obviousbox("setValue", "1");
$grid.datagrid("load", params);
```

HTML 中通过 `class="kui-xxx"` 和 `kui-options="..."` 声明组件。
