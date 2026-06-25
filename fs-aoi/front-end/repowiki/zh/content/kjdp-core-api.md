# @szkingdom/kjdp-core API 参考

## 目录

1. [包概述](#包概述)
2. [导入方式](#导入方式)
3. [request — HTTP 请求模块](#request--http-请求模块)
4. [useDict — 数据字典模块](#usedict--数据字典模块)
5. [useSysParam — 系统参数模块](#usesysparam--系统参数模块)
6. [useOrg — 机构数据模块](#useorg--机构数据模块)
7. [useSystem — 系统登录模块](#usesystem--系统登录模块)
8. [响应数据结构 FormattedResponse](#响应数据结构-formattedresponse)
9. [HttpConfig 配置项说明](#httpconfig-配置项说明)

---

## 包概述

`@szkingdom.kjdp/core` 是公司自研的前端核心基础包，提供统一的 HTTP 请求封装、数据字典、系统参数、机构数据查询等能力。所有业务开发均通过此包与后端服务交互。

**导出清单：**

| 导出名        | 类型          | 用途                                  |
| ------------- | ------------- | ------------------------------------- |
| `request`     | object        | HTTP 请求（最核心，90% 场景）         |
| `useDict`     | object        | 数据字典查询与翻译                    |
| `useSysParam` | object        | 系统公共参数查询                      |
| `useOrg`      | object        | 机构数据查询                          |
| `useSystem`   | namespace     | 登录/登出/获取当前用户信息            |
| `OrgType`     | enum          | 机构类型枚举                          |
| `RightType`   | enum          | 权限类型枚举                          |
| `axios`       | AxiosInstance | 底层 axios 实例（极少直接使用）       |
| `installer`   | function      | Vue 插件安装函数（在 main.js 中使用） |

---

## 导入方式

```js
import { request, useDict, useSysParam, useOrg, useSystem, OrgType, RightType } from '@szkingdom.kjdp/core'
```

---

## request — HTTP 请求模块

> **日常业务开发 90% 的场景使用 `request.service`**

### request.service（最常用）

调用后端 FS 协议服务接口。

```ts
request.service(serviceCode: string, data?: object, options?: Partial<HttpConfig>): Promise<FormattedResponse>
```

**参数说明：**

| 参数          | 类型                  | 必填 | 说明                          |
| ------------- | --------------------- | ---- | ----------------------------- |
| `serviceCode` | `string`              | 是   | 后端功能号，如 `'F010001001'` |
| `data`        | `object`              | 否   | 请求参数，默认 `{}`           |
| `options`     | `Partial<HttpConfig>` | 否   | 覆盖全局配置的请求选项        |

**返回值：** `Promise<FormattedResponse>`，见 [FormattedResponse 结构](#响应数据结构-formattedresponse)

**使用示例：**

```js
// 基础用法：查询列表
const res = await request.service('F010001001', { ORG_CODE: '001' })
const list = res.data0 // 第一个数据集（数组）

// 获取单行数据
const row = res.row // 第一条记录（对象）

// 判断成功（框架已自动处理，通常不需要手动判断）
// 框架会在 MSG_CODE !== '0' 时自动弹窗报错并 reject

// 自主处理错误（catchError 模式）
const res = await request.service('F010001001', {}, { catchError: true })
if (res.head.code !== '0') {
  // 自己处理错误
}

// 多数据集场景（后端返回多个 ANS_COMM_DATA）
const res = await request.service('F010001001', {})
const list1 = res.data0 // 第一个数据集
const list2 = res.data1 // 第二个数据集
const list3 = res.data2 // 第三个数据集
// data 是所有数据集的数组：[data0, data1, data2, ...]
```

---

### request.post

发送普通 POST 请求（非 FS 协议场景）。

```ts
request.post(url: string, data?: any, options?: Partial<HttpConfig>): Promise<any>
```

```js
const res = await request.post('/api/custom', { key: 'value' })
```

---

### request.get

发送 GET 请求。

```ts
request.get(url: string, options?: Partial<HttpConfig>): Promise<any>
```

```js
const res = await request.get('/api/config')
```

---

### request.all

并发执行多个请求（等同于 `Promise.all`）。

```ts
request.all(args: Promise[]): Promise<any[]>
```

```js
const [res1, res2] = await request.all([request.service('F010001001', {}), request.service('F010001002', {})])
```

---

### request.getConfig

获取当前全局 HttpConfig 配置。

```js
const config = request.getConfig()
```

---

## useDict — 数据字典模块

> 带本地缓存，相同字典码不会重复请求。

### 系统数据字典

#### useDict.getDict（最常用）

获取单个字典的全部字典项。

```ts
useDict.getDict(dictCode: string, subSys?: string): Promise<DictItem[]>
```

```js
// DictItem 结构：{ value: string, name: string, order: string, subSys: string, orgCode: string }
const items = await useDict.getDict('ID_TYPE')
// 结果示例：[{ value: '0', name: '身份证', ... }, { value: '1', name: '护照', ... }]
```

---

#### useDict.getDicts

批量获取多个字典。

```ts
useDict.getDicts(dictCodes: string | string[], subSys?: string): Promise<Record<string, Dict>>
```

```js
// 方式1：逗号分隔字符串
const dicts = await useDict.getDicts('ID_TYPE,USER_TYPE')
// 方式2：数组
const dicts = await useDict.getDicts(['ID_TYPE', 'USER_TYPE'])
// 结果：{ ID_TYPE: { code, name, items: [] }, USER_TYPE: { ... } }
```

---

#### useDict.getItem

获取字典中的某一个字典项。

```ts
useDict.getItem(dictCode: string, itemCode: string, subSys?: string): Promise<DictItem | undefined>
```

```js
const item = await useDict.getItem('ID_TYPE', '0')
// 结果：{ value: '0', name: '身份证', ... }
```

---

#### useDict.getItemText

获取字典项的显示名称（最常用的翻译场景）。

```ts
useDict.getItemText(dictCode: string, itemCode: string, subSys?: string): Promise<string | undefined>
```

```js
const text = await useDict.getItemText('ID_TYPE', '0')
// 结果：'身份证'
```

---

#### useDict.translate（重要）

批量翻译数据对象/数组中的字典字段，翻译结果写入 `字段名_TEXT`。

```ts
useDict.translate(data: object | object[], transConfig: string | Record<string, string>, subSys?: string): Promise<object | object[]>
```

**transConfig 的两种写法：**

```js
// 方式1：字符串（字段名 = 字典码）
// data.ID_TYPE 用 ID_TYPE 字典翻译，结果写入 data.ID_TYPE_TEXT
const result = await useDict.translate(data, 'ID_TYPE,USER_TYPE')

// 方式2：对象（字段名 → 字典码 映射，字段名 ≠ 字典码时使用）
// data.AGENT_ID_TYPE 用 ID_TYPE 字典翻译，结果写入 data.AGENT_ID_TYPE_TEXT
const result = await useDict.translate(data, {
  AGENT_ID_TYPE: 'ID_TYPE',
  USER_TYPE: 'USER_TYPE'
})

// 支持数组（每一项都翻译）
const translatedList = await useDict.translate(list, 'ID_TYPE,USER_TYPE')
```

---

#### useDict.translateByOrgDict

使用机构数据字典翻译（同 translate，多一个 orgCode 参数）。

```ts
useDict.translateByOrgDict(data, transConfig, orgCode: string, subSys?: string): Promise<object | object[]>
```

---

### 机构数据字典

> 与系统数据字典 API 对称，多一个 `orgCode` 参数。

```js
// 对应关系：
useDict.getOrgDict(dictCode, orgCode, subSys?)       // 对应 getDict
useDict.getOrgDicts(dictCodes, orgCode, subSys?)     // 对应 getDicts
useDict.getOrgItem(dictCode, itemCode, orgCode, subSys?)      // 对应 getItem
useDict.getOrgItemText(dictCode, itemCode, orgCode, subSys?)  // 对应 getItemText
```

---

### 缓存管理

```js
useDict.clearCache('ID_TYPE') // 清除指定系统字典缓存
useDict.clearCache() // 清除所有系统字典缓存
useDict.clearOrgCache('001', 'ID_TYPE') // 清除指定机构字典缓存
useDict.clearAllCache() // 清除所有缓存（系统+机构）
```

---

## useSysParam — 系统参数模块

> 带本地缓存。

### useSysParam.getValue（最常用）

获取单个系统参数的值（字符串）。

```ts
useSysParam.getValue(code: string, subSys?: string): Promise<string>
```

```js
const value = await useSysParam.getValue('MAX_UPLOAD_SIZE')
// 结果：'10485760'
```

---

### useSysParam.getData

获取单个系统参数的完整对象。

```ts
useSysParam.getData(code: string, subSys?: string): Promise<ParamData>
```

```js
// ParamData 结构：{ code, value, subSys, name, remark }
const param = await useSysParam.getData('MAX_UPLOAD_SIZE')
```

---

### useSysParam.getDatas

批量获取多个系统参数。

```ts
useSysParam.getDatas(codes: string | string[], subSys?: string): Promise<Record<string, ParamData>>
```

```js
// 方式1：逗号分隔字符串
const params = await useSysParam.getDatas('PARAM_A,PARAM_B')
// 方式2：数组
const params = await useSysParam.getDatas(['PARAM_A', 'PARAM_B'])
```

---

## useOrg — 机构数据模块

> 带本地缓存。

### 快捷方法（最常用）

```ts
useOrg.getIntOrg(rightType?: RightType): Promise<OrgItem[]>      // 内部机构
useOrg.getBankOrg(rightType?: RightType): Promise<OrgItem[]>     // 银行
useOrg.getExchangeOrg(rightType?: RightType): Promise<OrgItem[]> // 交易所/登记机构
useOrg.getFundOrg(rightType?: RightType): Promise<OrgItem[]>     // 基金公司
useOrg.getFuturesOrg(rightType?: RightType): Promise<OrgItem[]>  // 期货公司
```

**OrgItem 结构：**

```ts
{
  code: string // 机构代码 (ORG_CODE)
  name: string // 机构简称 (ORG_NAME)
  fullName: string // 机构全称 (ORG_FULL_NAME)
  parentOrg: string // 上级机构 (PAR_ORG)
  spell: string // 拼音 (ORG_SPELL)
  status: string // 状态 (ORG_STATUS)
  cls: string // 分类 (ORG_CLS)
  remark: string // 备注
  originalData: object // 原始数据
}
```

```js
// 获取有查询权限的内部机构
const orgList = await useOrg.getIntOrg()
// 获取有导出权限的内部机构
const orgList = await useOrg.getIntOrg(RightType.export)
```

---

### 枚举：OrgType

```ts
enum OrgType {
  internal = '0', // 内部机构
  bank = '1', // 银行
  exchange = '2', // 交易所/登记机构
  fund = '3', // 基金公司
  futures = '4' // 期货公司
}
```

---

### 枚举：RightType

```ts
enum RightType {
  add = '0', // 新增
  update = '1', // 修改
  delete = '2', // 删除
  query = '3', // 查询（默认）
  exec = '4', // 执行
  empower = '5', // 授权
  accept = '6', // 受理
  review = '7', // 审核
  manage = '8', // 管理
  check = '9', // 质检
  import = 'a', // 导入
  export = 'b', // 导出
  download = 'c', // 下载
  print = 'd', // 打印
  refresh = 's', // 刷新
  special = 'z' // 特殊
}
```

---

### useOrg.getOrg（底层方法）

```ts
useOrg.getOrg(orgType: OrgType, rightType: RightType): Promise<OrgItem[]>
```

---

## useSystem — 系统登录模块

> 主要由框架内部使用，业务开发偶尔用到 `getOpInfo`。

### useSystem.getOpInfo（常用）

获取当前登录用户信息。

```js
import { useSystem } from '@szkingdom.kjdp/core'

const opInfo = useSystem.getOpInfo()
// opInfo 结构（来自后端登录接口）：
// {
//   F_OP_USER: string,       // 操作员工号
//   F_OP_ROLE: string,       // 操作员角色
//   F_OP_BRANCH: string,     // 所属机构
//   LOG_IP: string,          // 登录 IP
//   USER_TICKET_INFO: string, // 会话票据
//   CURRENT_POST: string,    // 当前岗位
//   IRETCODE: '0',           // 登录成功标志
//   ...
// }
```

---

### useSystem.hasLogin

判断当前是否已登录。

```js
const isLoggedIn = useSystem.hasLogin() // returns boolean
```

---

### useSystem.login / useSystem.logout

```js
// 登录（框架内部使用）
await useSystem.login(params)

// 登出
await useSystem.logout()
```

---

## 响应数据结构 FormattedResponse

`request.service` 返回的标准化响应对象。

```ts
type FormattedResponse = {
  // 响应头信息
  head: {
    url: string // 请求的功能号/URL
    code: string // MSG_CODE，成功为 '0'
    message: string // MSG_TEXT，错误时为错误描述
    runtime: number // 服务端执行时长(ms)
    rows: number // DATA_ROWS，数据总行数（分页场景）
  }

  // 数据集（后端可返回多个数据集）
  data: any[][] // 所有数据集的数组：[data0, data1, ...]
  data0: any[] // 第一个数据集（最常用）
  data1: any[] // 第二个数据集
  data2: any[] // 第三个数据集，以此类推...
  row: any // data0[0]，第一条记录（单条查询场景常用）

  // 原始 axios 响应（极少需要）
  originResponse: AxiosResponse
}
```

**常见取值模式：**

```js
const res = await request.service('F010001001', {})

// 列表查询 → 取 data0
const list = res.data0

// 单条查询 → 取 row
const detail = res.row

// 多数据集 → 取 data0, data1, data2
const { data0: list, data1: summary } = res

// 分页总数 → 取 head.rows
const total = res.head.rows
```

---

## HttpConfig 配置项说明

`request.service` 的第三个参数 `options` 可覆盖全局配置，常用选项：

| 配置项               | 类型                 | 默认值         | 说明                                                |
| -------------------- | -------------------- | -------------- | --------------------------------------------------- |
| `catchError`         | `boolean`            | `false`        | `true` 时框架不自动弹窗，由业务代码自行处理错误     |
| `serviceBase`        | `string \| function` | `'/fsapi/'`    | 覆盖请求基础路径                                    |
| `baseURL`            | `string`             | 同 serviceBase | axios baseURL                                       |
| `timeout`            | `number`             | `120000`       | 请求超时时间(ms)                                    |
| `successCodes`       | `string[]`           | `['0']`        | 判断业务成功的 MSG_CODE 值                          |
| `responseDataFormat` | `boolean`            | `true`         | `false` 时跳过 FS 协议解析，直接返回原始 axios 响应 |
| `fsEncrypt`          | `boolean`            | `false`        | 是否加密请求体                                      |

**示例：**

```js
// 自主处理错误，不让框架弹窗
const res = await request.service('F010001001', {}, { catchError: true })

// 请求非默认服务地址
const res = await request.service('F010001001', {}, { serviceBase: '/otherapi/' })

// 超时时间改为 30 秒
const res = await request.service('F010001001', {}, { timeout: 30000 })
```
