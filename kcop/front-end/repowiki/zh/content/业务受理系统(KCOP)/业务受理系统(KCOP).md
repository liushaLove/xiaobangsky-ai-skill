# 业务受理系统

当前 copweb 中“AOI/业务受理”主要落在 `apps/opp/operator`、`apps/opp/review`、`apps/opp/manager` 等 legacy 页面，不是 Vue 的 `src/pages/aoi`。

核心模式：业务页面 HTML 引入 loader；JS 主模块导出 `exports.$ready`；流程节点用 `flow.init`；数据用 Backbone Model 或 service 获取；UI 用 KUI 表单、树、表格、弹窗。
