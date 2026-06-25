# HTTP 配置

当前 HTTP 不在 `src/config/http.js`，而在 `frame/core/ajax.js`。

`ajax.ajaxRequest` 会构造 KJDP 请求体：`REQ_MSG_HDR` 和 `REQ_COMM_DATA`。请求头中会补充操作员、机构、站点、session、服务号、客户机构等信息。

登录 token 从 sessionStorage 读取；响应头中的 `Authorization` 会写回 sessionStorage；401 或业务会话超时会触发统一提示。
