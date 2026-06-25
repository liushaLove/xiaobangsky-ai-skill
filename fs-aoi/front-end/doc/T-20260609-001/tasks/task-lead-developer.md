# 任务卡片 — Lead Developer

**任务 ID**：T-20260609-001-DEV
**模块**：uas / 复核申请查询
**类型**：新增功能
**优先级**：P1
**负责人**：Lead Developer
**关联需求**：T-20260609-001（复核申请查询菜单）
**关联设计**：参见 task-ui-designer.md 输出

---

## 开发目标

在指定路径创建复核申请查询页面，包含两个 Tab：

1. **复核申请查询**（主查询）
2. **复核申请明细**（点击行数据跳转）

**文件路径**：`src/pages/uas/views/business/ai-test/test.vue`

---

## 功能需求

### Tab 1：复核申请查询

| 项目     | 内容                                                                                                                                                                                                                                                                                                                                                                                       |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 查询条件 | 开始日期(BGN_DATE)、结束日期(END_DATE)、机构代码(INT_ORGES)、申请柜员(OP_CODE)                                                                                                                                                                                                                                                                                                             |
| 查询接口 | `F010001000`                                                                                                                                                                                                                                                                                                                                                                               |
| 固定入参 | `QUERY_MODE:'QUERY'`, `PAGE_COUNT:false`, `SUMMARY_FLAG:''`, `ARCHIVE_DATA_ZONE_CODE:''`, `F_FUNCTION:'970000060'`                                                                                                                                                                                                                                                                         |
| 展示列   | 记录序号(REC_NUM)、复核申请序号(CHK_APP_SN)、申请日期(CHK_APP_DATE)、申请时间(CHK_APP_TIME)、有效日期(EXP_DATE)、内部机构(INT_ORG)、功能代码(FUNC_CODE)、功能名称(FUNC_NAME)、复核组别(CHK_GRP)、复核级别(CHK_LVL)、已复核级别(CHKED_LVL)、复核状态(CHK_STATUS,dict)、复核申请柜员(OP_CODE)、申请柜员姓名(OP_NAME)、处理日期(PROC_DATE)、复核结果(PROC_RESULT,dict)、复核备注(PROC_REMARK) |
| 行为     | 双击/单击行 → 跳转 Tab2，传递 CHK_APP_SN                                                                                                                                                                                                                                                                                                                                                   |

### Tab 2：复核申请明细

| 项目     | 内容                                                                                                                                    |
| -------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| 触发     | 点击 Tab1 行数据自动切换                                                                                                                |
| 查询接口 | `F010001000`                                                                                                                            |
| 入参     | `QUERY_MODE:'SUBQUERY'`, `CHK_APP_SN:<选中行>`, `F_FUNCTION:'970000060'`                                                                |
| 展示列   | 复核申请序号(CHK_APP_SN)、复核日期(CHK_APP_DATE)、业务功能(LBM_DESC)、参数代码(LBM_PARA)、参数名称(LBM_PARA_NAME)、参数值(LBM_PARA_VAL) |

---

## 参考文件

- `src/pages/uas/views/business/recheck-manage/recheck-all-query.vue`（Tabs 组合模式）
- `src/pages/uas/views/business/recheck-manage/recheck-apply-query.vue`（QueryForm 用法）
- `src/pages/uas/views/business/recheck-manage/recheck-apply-detail.vue`（明细子组件）

---

## 技术规范

- 使用 `<script setup>` + Composition API
- Tab1 使用 `KuiGeneralView` + `QueryForm` + `TableView` + `Column` + `TablePagination`
- Tab2 使用 `KuiGeneralView` + `TableView`（data 属性接收数据）
- 外层使用 `KuiTabs` + `KuiTabPane`
- 接口调用使用 `request.service` from `@szkingdom.kjdp/core`
- 日期默认值：开始 = 上月今天，结束 = 今天
- 机构默认值：`opInfo.ORG_CODE`
- 样式只写必要的 Tab 自适应高度样式

---

## 完成定义（DoD）

- [x] 文件创建在指定路径
- [x] `npm run lint:fix` 通过
- [x] 查询条件、展示列、接口完全符合需求
- [x] 行点击跳转明细 Tab 正常
- [x] 代码风格与项目一致
