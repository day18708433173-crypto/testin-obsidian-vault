---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# Browser

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.Browser`

Web 浏览器信息上报。

## report

### 协议命令

```
{ "mkey": "script", "op": "Browser.report", "reqid": "xxx", "data": { "devices": [{ "type": "chrome", "version": "120.0", "protocol": "webdriver", ... }] } }
```

### 实现意图

上位机首次启动时上报名下所有浏览器信息（类型、版本、协议等）。协议字段（protocol）从第一项读取。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.devices | JSONArray | 是 | 浏览器列表 |
| data.devices[].type | String | — | 浏览器类型（chrome/firefox/edge 等） |
| data.devices[].version | String | — | 浏览器版本 |
| data.devices[].protocol | String | — | 通信协议（webdriver/devtools 等） |

### 调用链

```
trans.controller.req.script.Browser.report(Session, RequestContext)
  → ipcservice.browserReport(ucomid, browserList, protocol)
    → [real-cfg](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md) real_pc_browser
```

### 涉及表/SQL

- `real_pc_browser` — 浏览器信息表
