# 组件 API

## KUI 组件声明

```html
<input id="ORG_CODE"
       name="ORG_CODE"
       class="kui-combotree"
       kui-options="width:190,required:'true',req:[{service:'P000000200'}]" />
```

`kui-options` 是 legacy 字符串配置，维护时不要改成标准 JSON。

## 常用 API

| 组件 | 常用方法 |
| --- | --- |
| form | `form("load", data)`、`form("disable", fields)`、`form("validate")` |
| combobox | `combobox("getValue")`、`combobox("setValue", value)`、`combobox("loadData", rows)` |
| combotree | `combotree("getValue")`、`combotree("getValues")`、`combotree("setValues", arr)` |
| datagrid | `datagrid("load", params)`、`datagrid("getSelections")` |
| obviousbox | `obviousbox("getValue")`、`obviousbox("setValue", value)`、`obviousbox("setValues", values)` |
| dialog/window | `dialog("open")`、`window("destroy")` |
