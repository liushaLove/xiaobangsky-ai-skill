---
name: xml-to-general-view
description: 将旧版 XML 视图配置转换为 Vue3 KuiGeneralView 组件。当用户提供 XML 配置或说「转换 XML」「XML 转 Vue」「XML 转换为 GeneralView」时触发。支持 view、conf、qry_item、btn_item、col_item 等元素的完整转换。
---

# XML 转 KuiGeneralView 转换指南

将旧版 XML 视图配置转换为 Vue3 的 KuiGeneralView 组件。

---

## 转换流程

### 步骤 1：解析 XML 结构

识别以下 XML 元素：

- `<view>` - 视图容器
- `<conf>` - 配置信息
- `<qry_item>` - 查询条件
- `<btn_item>` - 操作按钮
- `<col_item>` - 表格列

### 步骤 2：生成 Vue 模板

按照 KuiGeneralView 标准结构生成代码：

```vue
<script setup>
// TODO: 自定义事件处理函数
</script>

<template>
  <KuiGeneralView>
    <QueryForm>
      <!-- 查询条件 -->
    </QueryForm>

    <TableView>
      <!-- 表格列 -->
    </TableView>

    <TableButtons>
      <!-- 操作按钮 -->
    </TableButtons>

    <TablePagination />
  </KuiGeneralView>
</template>

<style lang="scss" scoped></style>
```

---

## 元素映射规则

### 1. `<view>` → `<KuiGeneralView>`

| XML 属性 | Vue 属性 | 说明           |
| -------- | -------- | -------------- |
| `title`  | `title`  | 页面标题       |
| `id`     | —        | 通常不需要转换 |

**示例：**

```xml
<view id="YGT_UPM_SYSPARAM" title="公共参数设置">
```

↓

```vue
<KuiGeneralView title="公共参数设置"></KuiGeneralView>
```

---

### 2. `<conf>` → `<QueryForm>`

| XML 属性     | Vue 属性                     | 说明                              |
| ------------ | ---------------------------- | --------------------------------- |
| `service`    | `service`                    | 查询功能号                        |
| `req`        | `service` + `extended-param` | 复杂请求配置，提取 service 和参数 |
| `pagination` | —                            | 默认开启分页，无需转换            |
| `pageSize`   | —                            | 通过 TablePagination 配置         |
| `fit`        | —                            | 布局属性，默认处理                |
| `required`   | —                            | 必填校验标记                      |
| 事件属性     | 事件处理函数                 | 需要 script 中实现对应方法        |

#### conf 的 req 属性处理

当 conf 使用 `req` 而非 `service` 时：

```xml
<conf req="[{service:'YG000021',DICT_TYPE:'0',RIGHT_TYPE:'8'}]"/>
```

↓

```vue
<QueryForm service="YG000021" :extended-param="{ DICT_TYPE: '0', RIGHT_TYPE: '8' }"></QueryForm>
```

> 提取 req 中的 service 作为 QueryForm 的 service，其他参数放入 extended-param

#### conf 事件属性转换

| XML 属性       | Vue 处理方式          |
| -------------- | --------------------- |
| `onSelect`     | `@select` 事件        |
| `onBeforeLoad` | `:before-submit` 属性 |
| `onBeforeInit` | 需要在 script 中实现  |
| `onClickRow`   | `@row-click` 事件     |

**示例：**

```xml
<conf service="YG000001" onSelect="selectSysparam" onBeforeLoad="beforeLoadData"/>
```

↓

```vue
<QueryForm service="YG000001" :before-submit="beforeLoadData" @select="selectSysparam"></QueryForm>
```

```js
// script 中需要实现的方法
const selectSysparam = row => {
  // 原 onSelect 逻辑
}

const beforeLoadData = (done, params) => {
  // 原 onBeforeLoad 逻辑
  done()
  return params
}
```

---

### 3. `<qry_item>` → `<Item>`

| XML 属性        | Vue 属性              | 转换规则           |
| --------------- | --------------------- | ------------------ |
| `field`         | `prop`                | 直接映射           |
| `title`         | `label`               | 直接映射           |
| `type`          | `component` / `dict`  | 见类型映射表       |
| `edt_dict`      | `dict`                | 字典编码           |
| `edt_data`      | `:data`               | 转为数组格式       |
| `edt_validType` | `rules`               | 校验规则           |
| `dft_val`       | 放入 `extended-param` | 默认值             |
| `hidden="true"` | 放入 `extended-param` | 作为隐藏的固定参数 |

#### 隐藏字段处理

`hidden="true"` 的查询条件不生成 `<Item>`，而是通过 `QueryForm` 的 `:extended-param` 传递：

```xml
<qry_item field="RIGHT_TYPE" title="权限类型" dft_val="8" hidden="true"/>
```

↓

```vue
<QueryForm service="YG000001" :extended-param="{ RIGHT_TYPE: '8' }"></QueryForm>
```

#### 查询条件类型映射

| XML type    | Vue 处理方式                                      |
| ----------- | ------------------------------------------------- |
| `input`     | 默认，无需指定 component                          |
| `combobox`  | 优先使用 `edt_dict`，其次 `edt_data`              |
| `datebox`   | `component="date-box"`                            |
| `date`      | `component="date-box"`                            |
| `combotree` | `component="org-select"` 或自定义树选择组件       |
| `combogrid` | 需要自定义组件，添加 TODO 注释                    |
| `textarea`  | `type="textarea"`（用于 Item/Column）             |
| `numberbox` | 默认 input，可通过 rules 限制数字                 |
| `combobox`  | **若 edt_req 包含 P000000200，转换为 org-select** |

#### comboTree 处理（机构树选择）

**重要规则：调用 P000000200 接口的机构字段统一转换为 `org-select` 组件。**

org-select 组件基于 `useOrg` 方法，默认 `rightType=3`（查询权限）。根据业务场景需要设置不同的 `right-type` 值：

| right-type | 含义 | 适用场景         |
| ---------- | ---- | ---------------- |
| `3`        | 查询 | 默认值，查询场景 |
| `7`        | 审核 | 审核相关业务     |
| `8`        | 管理 | 管理类功能       |
| `6`        | 受理 | 受理类业务       |

**示例 1：基础机构选择（默认 right-type=3）**

```xml
<qry_item field="ORG_CODE" title="机构代码" type="combotree"
  edt_req="[{service:'P000000200',ORG_TYPE:'0',RIGHT_TYPE:'3'}]"
  edt_conf="{treeType: '1',nodeId:'ORG_CODE',nodeName:'ORG_CODE,ORG_FULL_NAME',parNode:'PAR_ORG'}"
  edt_multiple="true"/>
```

↓

```vue
<Item prop="ORG_CODE" label="机构代码" component="org-select" multiple />
```

**示例 2：带 right-type 参数（从 XML 提取）**

```xml
<qry_item field="ORG_CODE" title="机构代码" type="combotree"
  edt_req="[{service:'P000000200',ORG_TYPE:'0',RIGHT_TYPE:'8'}]"/>
```

↓

```vue
<Item prop="ORG_CODE" label="机构代码" component="org-select" right-type="8" />
```

**示例 3：银行机构选择（org-type=1）**

```xml
<qry_item field="BANK_ORG" title="银行机构" type="combotree"
  edt_req="[{service:'P000000200',ORG_TYPE:'1',RIGHT_TYPE:'3'}]"/>
```

↓

```vue
<Item prop="BANK_ORG" label="银行机构" component="org-select" org-type="1" />
```

> 注：从 `edt_req` 中提取 `RIGHT_TYPE` 作为 `right-type`，提取 `ORG_TYPE` 作为 `org-type`

**org-type 参数对应关系：**

| org-type | 含义            | useOrg 方法             |
| -------- | --------------- | ----------------------- |
| `0`      | 内部机构        | useOrg.getIntOrg()      |
| `1`      | 银行            | useOrg.getBankOrg()     |
| `2`      | 交易所/登记机构 | useOrg.getExchangeOrg() |
| `3`      | 基金公司        | useOrg.getFundOrg()     |
| `4`      | 期货公司        | useOrg.getFuturesOrg()  |

#### comboGrid 处理（表格选择器）

```xml
<qry_item field="OP_CODE" title="员工代码" type="combogrid"
  edt_req="[{service:'P001000111'}]" edt_valueField="OP_CODE" edt_textField="OP_CODE,USER_NAME"/>
```

↓

```vue
<!-- TODO: comboGrid 类型需要自定义组件实现 -->
<Item prop="OP_CODE" label="员工代码" />
```

#### combobox 特殊处理

**有 edt_dict：**

```xml
<qry_item field="PAR_LVL" title="参数组类型" type="combobox" edt_dict="PAR_CLS"/>
```

↓

```vue
<Item prop="PAR_LVL" label="参数组类型" dict="PAR_CLS" />
```

**有 edt_data：**

```xml
<qry_item field="MAINTAIN_FLAG" title="参数状态" type="combobox"
  edt_data="[{dict_des:'只读',dict_val:'0'},{dict_des:'允许修改',dict_val:'1'}]"
  edt_textField="dict_val,dict_des"/>
```

↓

```vue
<Item
  prop="MAINTAIN_FLAG"
  label="参数状态"
  :data="[
    { label: '只读', value: '0' },
    { label: '允许修改', value: '1' }
  ]"
/>
```

> 注：edt_textField 的第一个字段映射为 value，第二个映射为 label

#### edt_req 动态数据源

当 combobox 需要从接口获取选项时：

```xml
<qry_item field="SYS_CODE" title="子系统" type="combobox"
  edt_req="[{service:'F0900011',SYS_STAT:'1',RIGHT_TYPE:'8'}]"
  edt_valueField="SYS_CODE" edt_textField="SYS_CODE,SYS_NAME"/>
```

↓

```vue
<Item prop="SYS_CODE" label="子系统" service="F0900011" :service-param="{ SYS_STAT: '1', RIGHT_TYPE: '8' }" />
```

> 使用 `service` + `service-param` 替代 `edt_req`，`props` 指定字段映射

#### edt_multiple 多选处理

```xml
<qry_item field="CHANNEL" title="操作渠道" type="combobox" edt_dict="CHANNEL" edt_multiple="true"/>
```

↓

```vue
<Item prop="CHANNEL" label="操作渠道" dict="CHANNEL" multiple />
```

#### 校验规则转换

| XML edt_validType     | Vue rules                            |
| --------------------- | ------------------------------------ |
| `length[1,32]`        | `rules="length[1,32]"`               |
| `length[1,64]`        | `rules="length[1,64]"`               |
| `val[1,32]`           | `rules="length[1,32]"`               |
| `num[1,16]`           | `rules="num[1,16]"`                  |
| `num[0,4]`            | `rules="num[0,4]"`                   |
| `numberex[10,16]`     | `rules="num[1,16]"`                  |
| `numchar[16,32]`      | 自定义校验或 `rules="length[16,32]"` |
| `zint[18]`            | 正整数校验                           |
| `begDate['END_DATE']` | 自定义日期范围校验                   |
| `endDate['BEG_DATE']` | 自定义日期范围校验                   |

---

### 4. `<btn_item>` → `<TableButton>` / `<KuiButton>`

#### 标准 CRUD 按钮

根据 `handler` 属性判断按钮类型：

| handler 值     | Vue type              |
| -------------- | --------------------- |
| `commonAdd`    | `type="insert"`       |
| `commonModify` | `type="update"`       |
| `commonDelete` | —（使用 delete 类型） |

**新增按钮：**

```xml
<btn_item text="新增" service="YG000002" handler="commonAdd" title="系统参数新增"/>
```

↓

```vue
<TableButton service="YG000002" type="insert" />
```

**删除按钮：**

```xml
<btn_item text="删除" service="YG000004" handler="commonDelete" title="删除"/>
```

↓

```vue
<TableButton service="YG000004" type="delete" />
```

#### btn_item 事件属性转换

| XML 属性            | Vue 处理方式                 |
| ------------------- | ---------------------------- |
| `onBeforeOpen`      | `:before-dialog-open`        |
| `onBeforeSave`      | `:before-submit`             |
| `onSaveSuccess`     | `@success`                   |
| `onBeforeLoad`      | `:before-load`               |
| `onBeforeDelete`    | `:before-delete`             |
| `onDeleteSuccess`   | `@delete-success`            |
| `onFormInitSuccess` | 需要自定义处理               |
| `panelHeight`       | 通过 `dialogProp.width` 配置 |
| `panelWidth`        | 通过 `dialogProp.width` 配置 |

**示例：**

```xml
<btn_item text="新增" service="YG000002" handler="commonAdd"
  onBeforeOpen="formBeforeOpen" onBeforeSave="sysParamBeforeSave" panelHeight="450"/>
```

↓

```vue
<TableButton
  service="YG000002"
  type="insert"
  :before-dialog-open="formBeforeOpen"
  :before-submit="sysParamBeforeSave"
/>
```

> 注：panelHeight/panelWidth 在 Vue3 中通过父组件的 dialogProp 统一配置

#### 自定义按钮

非标准 CRUD 按钮生成 `<KuiButton>` 并添加 TODO 注释：

```xml
<btn_item text="同步" title="同步子系统参数" handler="sysSyncBtnHandler" iconCls="icon-execute"/>
<btn_item text="刷新缓存" title="刷新一柜通参数缓存" handler="refreshParamCache"/>
```

↓

```vue
<KuiButton @click="handleSync">同步</KuiButton>
<KuiButton @click="handleRefreshCache">刷新缓存</KuiButton>
```

```js
// TODO: 实现同步子系统参数功能
const handleSync = () => {
  // 原 handler: sysSyncBtnHandler
}

// TODO: 实现刷新缓存功能
const handleRefreshCache = () => {
  // 原 handler: refreshParamCache
}
```

---

### 5. `<col_item>` → `<Column>`

| XML 属性              | Vue 属性             | 转换规则       |
| --------------------- | -------------------- | -------------- |
| `field`               | `prop`               | 直接映射       |
| `title`               | `label`              | 直接映射       |
| `width`               | `width`              | 直接映射       |
| `type`                | `component` / `dict` | 见类型映射表   |
| `edt_dict`            | `dict`               | 字典编码       |
| `edt_data`            | `:data`              | 转为数组格式   |
| `edt_required="true"` | `required`           | 必填标记       |
| `edit_flag`           | `disabled`           | 见编辑标志映射 |
| `listHidden="true"`   | 省略该列             | 不在列表中显示 |

#### 编辑标志（edit_flag）映射

| edit_flag | 含义             | Vue 属性                           |
| --------- | ---------------- | ---------------------------------- |
| `1`       | 可新增不可修改   | `disabled="update"`                |
| `2`       | 不可新增不可修改 | `disabled`                         |
| `3`       | 不显示           | 不生成该列（或用 `hidden-in-all`） |
| `4`       | 不可新增可修改   | `disabled="insert"`                |

**示例：**

```xml
<col_item field="SYS_CODE" title="子系统" edit_flag="1" edt_required="true"/>
```

↓

```vue
<Column prop="SYS_CODE" label="子系统" disabled="update" required />
```

#### 列类型映射

| XML type    | Vue 处理方式                                       |
| ----------- | -------------------------------------------------- |
| `input`     | 默认，无需指定                                     |
| `combobox`  | 优先使用 `edt_dict`，其次 `edt_data`               |
| `textarea`  | `type="textarea"`（用于 Column）                   |
| `numberbox` | 默认 input，数字右对齐                             |
| `datebox`   | 日期格式展示                                       |
| `checkbox`  | `type="selection"` 或自定义                        |
| `combogrid` | 需要自定义组件                                     |
| `combotree` | `component="org-select"`，提取 right-type/org-type |
| `combobox`  | **若 edt_req 包含 P000000200，转换为 org-select**  |

#### edt_req 动态数据源（列）- P000000200 机构接口特殊处理

**重要：若 edt_req 包含 P000000200 接口，应转换为 `org-select` 组件：**

```xml
<col_item field="OP_ORG" title="操作机构" type="combobox"
  edt_req="[{service:'P000000200',ORG_TYPE:'0',RIGHT_TYPE:'3'}]"
  edt_textField="ORG_CODE,ORG_FULL_NAME" edt_valueField="ORG_CODE"/>
```

↓

```vue
<Column prop="OP_ORG" label="操作机构" component="org-select" />
```

> 注：P000000200 为机构查询接口，统一转换为 org-select 组件，无需 service/service-param

**非 P000000200 接口的 combobox：**

```xml
<col_item field="SYS_CODE" title="子系统" type="combobox"
  edt_req="[{service:'F0900011',SYS_STAT:'1'}]"
  edt_textField="SYS_CODE,SYS_NAME" edt_valueField="SYS_CODE"/>
```

↓

```vue
<Column prop="SYS_CODE" label="子系统" service="F0900011" :service-param="{ SYS_STAT: '1' }" />
```

#### edt_defaultValue 默认值

```xml
<col_item field="SYS_CODE" title="子系统" edt_defaultValue="99"/>
```

↓

```vue
<Column prop="SYS_CODE" label="子系统" :default-value="'99'" />
```

#### frozen 冻结列

```xml
<col_item field="REC_NUM" title="记录序号" frozen="1"/>
```

↓

```vue
<Column prop="REC_NUM" label="记录序号" fixed />
```

> frozen="1" 转换为 fixed 属性

#### align 对齐方式

```xml
<col_item field="RECORD" title="记录数量" align="right"/>
```

↓

```vue
<Column prop="RECORD" label="记录数量" align="right" />
```

#### sortType 排序类型

```xml
<col_item field="DICT_ORD" title="取值顺序" sortType="number"/>
```

↓

```vue
<Column prop="DICT_ORD" label="取值顺序" sortable />
```

> sortType 表示该列支持排序，转换为 sortable

---

## 完整转换示例

### XML 输入

```xml
<view id="YGT_UPM_SYSPARAM" title="公共参数设置">
    <conf service="YG000001" />
    <qry_item field="SYS_CODE" title="子系统" type="combobox" edt_dict="SYS_CODE"/>
    <qry_item field="PAR_NAME" title="参数名称" type="input" edt_validType="length[1,64]"/>
    <qry_item field="RIGHT_TYPE" title="权限类型" dft_val="8" hidden="true"/>

    <btn_item text="新增" service="YG000002" handler="commonAdd"/>
    <btn_item text="修改" service="YG000003" handler="commonModify"/>
    <btn_item text="同步" handler="sysSyncBtnHandler"/>

    <col_item field="SYS_CODE" title="子系统" width="80" edit_flag="1" edt_required="true"/>
    <col_item field="PAR_NAME" title="参数名称" width="150" edt_required="true"/>
    <col_item field="PAR_VAL" title="参数值" width="80"/>
</view>
```

### Vue 输出

```vue
<script setup>
// TODO: 实现同步功能
const handleSync = () => {
  // 原 handler: sysSyncBtnHandler
}
</script>

<template>
  <KuiGeneralView title="公共参数设置">
    <QueryForm service="YG000001" :extended-param="{ RIGHT_TYPE: '8' }">
      <Item prop="SYS_CODE" label="子系统" dict="SYS_CODE" />
      <Item prop="PAR_NAME" label="参数名称" rules="length[1,64]" />
    </QueryForm>

    <TableView>
      <Column prop="SYS_CODE" label="子系统" width="80" disabled="update" required />
      <Column prop="PAR_NAME" label="参数名称" width="150" required />
      <Column prop="PAR_VAL" label="参数值" width="80" />
    </TableView>

    <TableButtons>
      <TableButton service="YG000002" type="insert" />
      <TableButton service="YG000003" type="update" />
      <KuiButton @click="handleSync">同步</KuiButton>
    </TableButtons>

    <TablePagination />
  </KuiGeneralView>
</template>

<style lang="scss" scoped></style>
```

---

## 注意事项

### 代码规范

1. **子组件间空一行**：QueryForm / TableView / TableButtons / TablePagination 之间空行分隔
2. **省略默认属性**：`trigger="auto"`、`ref`、默认 width 等不显式声明
3. **组件无需注册**：KuiGeneralView 及子组件已全局注册，无需 import

### 转换注意

1. **service 必须保留**：功能号是后端接口标识，必须准确转换
2. **字典优先**：combobox 类型优先使用 `edt_dict`，无字典时才用 `edt_data`
3. **自定义按钮注释**：非标准按钮添加 TODO 注释说明原 handler 名称
4. **隐藏字段**：`hidden="true"` 的查询条件放入 `extended-param`，不生成 Item

### 需要用户确认的内容

转换完成后，提示用户确认：

1. 功能号（service）是否正确
2. 自定义按钮的事件实现
3. edit_flag=3 的列是否真的需要隐藏
4. 字典编码是否存在
5. combogrid 类型需要自定义组件实现
6. 原 XML 中的 formatter 是否需要保留（Vue3 中通过插槽实现）

---

## 特殊场景处理

### render="sform" 纯表单视图

当 view 的 render 属性为 `sform` 时，不生成 QueryForm 和 TableView，而是使用 `KuiGeneralForm`：

```xml
<view id="generalCuacctForm" render="sform" title="资金账号信息">
    <col_item field="USER_CODE" title="用户代码" type="input"/>
    <col_item field="CUACCT_CLS" title="资金账号类别" type="combobox" edt_dict="CUACCT_CLS"/>
</view>
```

↓

```vue
<KuiGeneralForm>
  <KuiInput prop="USER_CODE" label="用户代码" />
  <KuiSelect prop="CUACCT_CLS" label="资金账号类别" dict="CUACCT_CLS" />
</KuiGeneralForm>
```

### 主从表结构

当存在主从两个 view 时（如字典管理和字典项管理），使用 `:sub-view` 关联：

```vue
<KuiGeneralView ref="mainView">
  <!-- 主表内容 -->
</KuiGeneralView>

<KuiGeneralView :sub-view="mainView">
  <!-- 从表内容 -->
</KuiGeneralView>
```

### 树形选择器自定义

combotree 类型如果用于非机构选择场景，需要自定义组件：

```vue
<!-- TODO: 需要自定义树选择组件 -->
<Item prop="ORG_CODE" label="机构代码" />
```

### 日期默认值处理

XML 中日期默认值使用特殊格式：

```xml
<qry_item field="OCCUR_BEG_DATE" type="datebox" edt_defaultValue=":D0"/>
```

↓

```vue
<Item prop="OCCUR_BEG_DATE" label="开始日期" component="date-box" :default-value="dayjs().format('YYYYMMDD')" />
```

> `:D0` 表示当天日期，`:D-7` 表示7天前，需使用 dayjs 转换

### textarea 类型处理

XML 中的 textarea 类型转换为 Vue 的 `type="textarea"` 属性：

**查询条件（Item）：**

```xml
<qry_item field="REMARK" title="备注" type="textarea"/>
```

↓

```vue
<Item prop="REMARK" label="备注" type="textarea" />
```

**表格列（Column）：**

```xml
<col_item field="REMARK" title="备注" type="textarea" width="300"/>
```

↓

```vue
<Column prop="REMARK" label="备注" type="textarea" width="300" />
```

> 注：textarea 使用 `type="textarea"` 而非 `component` 属性
