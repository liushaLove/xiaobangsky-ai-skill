---
name: kui-general-view
description: 使用 @szkingdom.kjdp/ui 的 KuiGeneralView 组件开发通用增删改查页面。当用户说「帮我生成一个通用页面」「通用查询页」「通用增删改查」「GeneralView 页面」「查询列表页」，或描述「查询条件有 XXX，结果展示列有 XXX」时触发。涵盖 KuiGeneralView、QueryForm、Item、TableView、Column、TableButtons、TableButton、TablePagination、TableDialog 的完整用法与常用字段速查。
---

# KuiGeneralView 通用增删改查

> 所有组件已在 `createApp` 时全局注册，无需手动 import 注册，直接在模板中使用即可。

---

## 典型场景示例

### 场景 1：增删改查（最常见）

```html
<KuiGeneralView>
  <QueryForm service="P000000001">
    <Item prop="PAR_CODE" label="参数代码" />
    <Item prop="PAR_NAME" label="参数名称" />
    <Item prop="STATUS" label="状态" dict="STATUS" multiple />
  </QueryForm>

  <TableView>
    <Column prop="PAR_CODE" label="参数代码" />
    <Column prop="PAR_NAME" label="参数名称" />
    <Column prop="STATUS" label="状态" dict="STATUS" />
  </TableView>

  <TableButtons>
    <TableButton service="P000000002" type="insert" />
    <TableButton service="P000000003" type="update" />
    <TableButton service="P000000004" type="delete" />
  </TableButtons>

  <TablePagination />
</KuiGeneralView>
```

### 场景 2：手动查询 + 扩展参数

```js
import { ref } from 'vue'
import dayjs from 'dayjs'
import { useOpInfo } from '@szkingdom.kjdp/core'

const { opInfo } = useOpInfo()

const handleBeforeQuery = params => {
  // 查询前处理，返回 false 可中断
  return true
}

const handleRowDblclick = row => {
  // 双击行处理
}
```

```html
<KuiGeneralView>
  <QueryForm
    trigger="manual"
    service="F010662001"
    label-width="140"
    :extended-param="{ QUERY_MODE: 'QUERY', F_FUNCTION: '981000080' }"
    :before-submit="handleBeforeQuery"
  >
    <Item
      prop="BGN_DATE"
      label="开始日期"
      component="date-picker"
      required
      :default-value="dayjs().format('YYYYMMDD')"
    />
    <Item
      prop="END_DATE"
      label="结束日期"
      component="date-picker"
      required
      :default-value="dayjs().format('YYYYMMDD')"
    />
    <Item prop="INT_ORGES" label="操作机构" component="org-select" :default-value="opInfo.ORG_CODE" multiple />
  </QueryForm>

  <TableView @row-dblclick="handleRowDblclick">
    <Column prop="REC_NUM" label="记录序号" />
    <Column prop="SERIAL_NO" label="流水序号" />
    <Column prop="BIZ_DATE" label="业务日期" />
    <Column prop="INT_ORG" label="操作机构" component="org-select" />
  </TableView>

  <TablePagination />
</KuiGeneralView>
```

### 场景 3：纯展示表格（传入 data，无查询）

```js
import { ref } from 'vue'

const summaryData = ref([])
```

```html
<KuiGeneralView title="汇总">
  <TableView :data="summaryData">
    <Column prop="INT_ORG" label="机构" component="org-select" />
    <Column prop="RECORD" label="笔数" />
  </TableView>

  <TablePagination />
</KuiGeneralView>
```

### 场景 4：自定义操作列（行内按钮）

```html
<KuiGeneralView>
  <TableView>
    <Column type="oper" label="操作" fixed="right">
      <template #default="{ row }">
        <KuiButton type="primary" size="small" plain @click="handleOper(row)">续办</KuiButton>
      </template>
    </Column>
    <Column prop="BUSI_NAME" label="业务名称" />
  </TableView>

  <TablePagination />
</KuiGeneralView>
```

### 场景 5：通过 ref 获取选中行

```html
<KuiGeneralView ref="viewRef">
  <TableView multiple>
    <Column prop="BUSI_NAME" label="业务名称" />
  </TableView>

  <TablePagination />
</KuiGeneralView>
```

```js
const viewRef = ref(null)

const handleGetSelected = () => {
  const selected = viewRef.value.getSelectionData()
  // multiple=true 时返回数组，否则返回当前行对象
}
```

---

## 常用 QueryForm Item 清单

直接复制使用，需要时引入 `useOpInfo`：

```js
import { useOpInfo } from '@szkingdom.kjdp/core'
import { dayjs } from '@szkingdom.kjdp/ui'
const { opInfo } = useOpInfo()
```

```html
<!-- 时间范围 -->
<Item prop="START_TIME" label="开始时间" component="date-box" />
<Item prop="END_TIME" label="结束时间" component="date-box" :default-value="dayjs().format('YYYYMMDD')" />

<!-- 日期范围 -->
<Item prop="BGN_DATE" label="开始日期" component="date-box" />
<Item prop="END_DATE" label="结束日期" component="date-box" :default-value="dayjs().format('YYYYMMDD')" />

<!-- 机构选择（默认当前登录机构） -->
<Item prop="INT_ORG" label="机构编码" component="org-select" :default-value="opInfo.ORG_CODE" multiple />

<!-- 字典选择 -->
<Item prop="ID_TYPE" label="证件类型" dict="ID_TYPE" multiple />
<Item prop="USER_TYPE" label="用户类型" dict="USER_TYPE" multiple />
```

---

## 常用 TableView Column 清单

```html
<!-- 客户基本信息 -->
<Column prop="CUST_FNAME" label="客户全称" width="200" />
<Column prop="USER_TYPE" label="客户类型" dict="USER_TYPE" width="80" />
<Column prop="SEX" label="性别" dict="SEX" width="80" />
<Column prop="BIRTHDAY" label="出生日期" width="120" />
<Column prop="EDUCATION" label="学历" dict="EDUCATION" width="120" />
<Column prop="OCCU_TYPE" label="职业" dict="OCCU_EXTYPE_NEW" width="120" />
<Column prop="NATIONAL_ATTR" label="国有属性" dict="NATIONAL_ATTR" width="120" />

<!-- 证件信息 -->
<Column prop="ID_TYPE" label="证件类型" dict="ID_TYPE" width="120" />
<Column prop="ID_CODE" label="证件号码" width="200" />
<Column prop="ID_EXP_DATE" label="证件有效期" width="100" />

<!-- 机构 -->
<Column prop="ORG_CODE" label="机构代码" component="org-select" width="240" />
<Column prop="INT_ORG" label="机构代码" component="org-select" width="240" />

<!-- 账户 -->
<Column prop="TRDACCT" label="交易账户" width="120" />
<Column prop="YMT_CODE" label="一码通账号" width="120" />

<!-- 其他 -->
<Column prop="CHANNELS" label="操作渠道" dict="CHANNEL" multiple seperator="" width="300" />
<Column prop="REMARK" label="备注信息" width="300" />
```

---

## KuiGeneralView Props

| 属性          | 类型    | 默认值  | 说明                                                            |
| ------------- | ------- | ------- | --------------------------------------------------------------- |
| title         | String  | `''`    | 页面标题                                                        |
| showTitle     | Boolean | `false` | 是否显示标题                                                    |
| showReturnAll | Boolean | `false` | 是否显示返回全部按钮                                            |
| showRowDetail | Boolean | `false` | 是否显示行详情                                                  |
| showRefresh   | Boolean | `true`  | 是否显示刷新按钮                                                |
| showTool      | Boolean | `false` | 是否显示工具栏（列配置/导出等）                                 |
| toolMenus     | Array   | `[]`    | 工具栏菜单项                                                    |
| exportProp    | Object  | `null`  | 导出配置                                                        |
| confirmType   | String  | —       | 删除确认方式：`pop`（气泡）/ `dialog`（弹窗）/ `''`（直接执行） |
| dialogProp    | Object  | `{}`    | 全局弹窗配置                                                    |
| direction     | String  | `'row'` | 子组件排列方向                                                  |

## KuiGeneralView 方法（通过 ref 调用）

| 方法                 | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| `getSelectionData()` | 获取选中行数据；multiple=true 时返回数组，否则返回当前行对象 |

---

## QueryForm Props

| 属性              | 类型          | 默认值    | 说明                                               |
| ----------------- | ------------- | --------- | -------------------------------------------------- |
| service           | String        | `''`      | 查询功能号（接口编号）                             |
| trigger           | String        | `'auto'`  | 触发方式：`auto`（自动查询）/ `manual`（手动触发） |
| extendedParam     | Object        | `{}`      | 扩展请求参数                                       |
| beforeSubmit      | Function      | `null`    | 查询前拦截，返回 false 可中断查询                  |
| onSuccess         | Function      | `null`    | 查询成功回调                                       |
| onError           | Function      | `null`    | 查询失败回调                                       |
| labelWidth        | String/Number | `'120px'` | 标签宽度                                           |
| inputWidth        | String/Number | `'200px'` | 输入框宽度                                         |
| collapsed         | Boolean       | `true`    | 是否显示折叠图标                                   |
| defaultCollapsed  | Boolean       | `true`    | 是否默认收起                                       |
| hideNum           | Number/String | `'auto'`  | 超过多少列时隐藏剩余条件                           |
| enableEnterSearch | Boolean       | `false`   | 是否支持回车触发查询                               |
| v-model           | Object        | —         | 绑定查询表单数据（可选）                           |

---

## Item Props（QueryForm / Column 通用）

| 属性                         | 类型           | 默认值                          | 说明                                                    |
| ---------------------------- | -------------- | ------------------------------- | ------------------------------------------------------- |
| prop                         | String         | —                               | 字段名（必填）                                          |
| label                        | String         | —                               | 标签文本                                                |
| component                    | String         | —                               | 组件类型，见下方列表                                    |
| required                     | Boolean        | —                               | 是否必填                                                |
| rules                        | String         | —                               | 校验规则，如 `length[1,48]`、`num[1,16]`                |
| rulesCallback                | Function       | —                               | 自定义校验函数                                          |
| dict                         | String         | —                               | 字典代码，自动加载选项                                  |
| data                         | Array          | —                               | 手动传入选项数据                                        |
| service                      | String         | —                               | 通过功能号加载选项                                      |
| serviceParam                 | Object         | —                               | service 请求附加参数                                    |
| props                        | Object         | `{value:'value',label:'label'}` | 选项 value/label 字段映射，也可用 `valueKey`/`labelKey` |
| defaultValue / default-value | any            | —                               | 默认值                                                  |
| multiple                     | Boolean        | —                               | 是否多选                                                |
| hidden                       | String/Boolean | —                               | 在新增/修改弹窗中隐藏，值：`insert`/`update`/`true`     |
| disabled                     | String/Boolean | —                               | 在新增/修改弹窗中禁用，值：`insert`/`update`/`true`     |
| split                        | Number         | `1`                             | 一行拆分为几列                                          |
| labelWidth                   | Number/String  | —                               | 单独设置该项标签宽度                                    |

### component 可选值

| 值              | 说明                      |
| --------------- | ------------------------- |
| `input`（默认） | 文本输入框                |
| `select`        | 下拉选择                  |
| `date-picker`   | 日期选择（格式 YYYYMMDD） |
| `date-box`      | 日期输入框                |
| `org-select`    | 机构选择器                |
| `textarea`      | 多行文本                  |

---

## TableView Props

| 属性       | 类型     | 默认值  | 说明                                |
| ---------- | -------- | ------- | ----------------------------------- |
| multiple   | Boolean  | `false` | 是否多选                            |
| rowKey     | String   | —       | 行唯一键字段名                      |
| maxHeight  | String   | `''`    | 表格最大高度                        |
| useVirtual | Boolean  | `false` | 是否开启虚拟滚动                    |
| border     | Boolean  | `true`  | 是否显示边框                        |
| beforeLoad | Function | `null`  | 数据加载前拦截                      |
| afterLoad  | Function | `null`  | 数据加载后回调                      |
| data       | Array    | —       | 手动传入数据（不走 QueryForm 请求） |

## TableView Events

`select` / `select-all` / `selection-change` / `row-click` / `row-dblclick` / `cell-click` / `sort-change` / `current-change` / `dialogOpen` / `dialogClose`

---

## Column Props

| 属性                     | 类型           | 默认值                          | 说明                                          |
| ------------------------ | -------------- | ------------------------------- | --------------------------------------------- |
| prop                     | String         | —                               | 字段名（必填）                                |
| label                    | String         | —                               | 列标题                                        |
| width                    | String/Number  | —                               | 列宽                                          |
| fixed                    | String/Boolean | —                               | 固定列：`true`/`left`/`right`                 |
| isHidden / hidden-in-all | Boolean        | `false`                         | 是否隐藏列                                    |
| dict                     | String         | —                               | 字典代码，列值自动翻译                        |
| component                | String         | —                               | 列渲染组件类型（同 Item component）           |
| props                    | Object         | `{value:'value',label:'label'}` | 同 Item props                                 |
| type                     | String         | —                               | 特殊列类型：`index`（序号）/ `oper`（操作列） |

### Column 自定义插槽

```html
<Column prop="STATUS" label="状态" width="120">
  <template #default="{ row, column, $index }">
    <span>{{ row.STATUS }}</span>
  </template>
</Column>
```

---

## TableButton Props

| 属性          | 类型             | 默认值  | 说明                                 |
| ------------- | ---------------- | ------- | ------------------------------------ |
| type          | String           | —       | 按钮类型：`insert`/`update`/`delete` |
| service       | String           | `''`    | 操作功能号                           |
| extendedParam | Object           | `{}`    | 附加请求参数                         |
| beforeSubmit  | Function         | `null`  | 提交前拦截                           |
| onSuccess     | Function         | `null`  | 操作成功回调                         |
| buttonText    | String           | —       | 自定义按钮文本                       |
| disabled      | Boolean/Function | `false` | 是否禁用                             |
| confirmType   | String           | —       | 覆盖父级 confirmType                 |
| inline        | Boolean          | —       | 是否在行内显示（操作列按钮）         |
| funcCode      | String           | `''`    | 权限控制码                           |

---

## TablePagination Props

| 属性       | 类型    | 默认值                 | 说明                     |
| ---------- | ------- | ---------------------- | ------------------------ |
| page-sizes | Array   | `[20,50,100,500,1000]` | 每页条数选项             |
| autoScroll | Boolean | `true`                 | 翻页后是否自动滚动到顶部 |
| isRpcPage  | Boolean | `false`                | 是否 RPC 增量分页        |

---

## TableDialog Props

| 属性              | 类型     | 默认值 | 说明             |
| ----------------- | -------- | ------ | ---------------- |
| dialogWidth       | Number   | `600`  | 弹窗宽度（px）   |
| showFooter        | Boolean  | `true` | 是否显示底部按钮 |
| formProp          | Object   | `{}`   | 弹窗内表单配置   |
| beforeDialogOpen  | Function | `null` | 弹窗打开前拦截   |
| beforeDialogClose | Function | `null` | 弹窗关闭前拦截   |

---

## 注意事项

1. **组件顺序**：`QueryForm` → `TableView` → `TableButtons` → `TablePagination`，顺序影响布局渲染；子组件之间空一行。
2. **省略默认属性**：`trigger`、`ref`、`width`、`fixed`、`show-row-detail`、`show-return-all`、`show-tool` 均有全局默认配置，不需要单独控制时不写。
3. **功能号**：`service` 填写后端接口功能号（如 `F010001000`），`extendedParam` 传递附加业务参数。
4. **字典翻译**：`Column` 加 `dict` 属性可自动翻译字段值，无需手动 formatter。
5. **trigger**：日期范围查询等需要用户填完再查的页面显式写 `trigger="manual"`；默认自动查询不写。
6. **Column width**：默认不写，全局有统一配置；固定宽度字段（如序号列）才单独指定。
7. **多选**：`TableView` 加 `multiple` 后，通过 `viewRef.value.getSelectionData()` 获取多选数据数组。
8. **自定义按钮**：`TableButtons` 支持混入自定义 `KuiButton`，不必全用 `TableButton`。
9. **clearable 默认开启**：全局已配置 `clearable=true`，不需要单独写。
