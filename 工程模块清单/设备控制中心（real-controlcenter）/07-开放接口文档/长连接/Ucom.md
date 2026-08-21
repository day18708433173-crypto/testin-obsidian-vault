---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# Ucom

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.Ucom`

上位机交互相关命令，包括上位机间通知转发、上位机信息同步、上位机告警、探头配置。

## notify

### 协议命令

上位机侧发送：

```
{ "mkey": "script", "op": "Ucom.notify", "reqid": "xxx", "data": { "targetUcomid": "target@testin.cn", "content": {...} } }
```

平台侧（上位机收到后返回）：

```
{ "mkey": "script", "op": "UcomControl.handleNotice", "code": 0, "data": { ... }, "resid": "xxx" }
```

### 实现意图

上位机间消息通知转发。上位机 A 通过 `Ucom.notify` 向控制中心发送消息，控制中心通过 `IProtocolService.add` 将消息以 `UcomControl.handleNotice` 下发到目标上位机 B。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.targetUcomid | String | 是 | 目标上位机账号 |
| data.content | JSONObject | 是 | 转发内容 |

### 调用链

```
trans.controller.req.script.Ucom.notify(Session, RequestContext)
  → iprotocolservice.add(MODULE_NODE_ID, activity, targetUcomid, null, "script", "UcomControl.handleNotice", textContent, pendingRequest, null, null)
    → Redis 协议队列 → ProtocolDispatchThread → 推送至目标上位机
```

### 关键代码摘录

```java
// 构建下发报文
jsonContent.put("op", "Ucom.handleNotice");
JSONObject data = new JSONObject();
data.put("sourceUcomid", ucomid);  // 生产者
data.put("content", content);
jsonContent.put("data", data);

// 写入协议队列，异步分发给目标上位机
iprotocolservice.add(Config.MODULE_NODE_ID, type, sessionKey, reqid, mkey, op, textContent, status, null, null);
```

注意：`UcomControl` 类在 trans.controller.res 包下**不存在**（不在 设备控制中心 模块内），但 `ProtocolServiceImpl` 会尝试反射加载。当类不存在时，走 `invalidMethod` 分支，返回成功（`result: 1`）以完成下发协议的 `finish` 状态更新。

---

## syncInfo

### 协议命令

```
{ "mkey": "script", "op": "Ucom.syncInfo", "reqid": "xxx", "data": { "ucomInfo": { "ucomid": "...", ... } } }
```

### 实现意图

上位机信息同步上报（位置、状态等）。从 `PcCfgPool` 内存中补充 caseFloor 和 caseNum 信息。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.ucomInfo | JSONObject | 是 | 上位机信息 JSON（含 ucomid 等） |

### 调用链

```
trans.controller.req.script.Ucom.syncInfo(Session, RequestContext)
  → PcCfgPoolUtil.getPcCfgPool().get(ucomid)    // 内存获取机柜位置
  → iUcomInfoService.add(UcomInfo)                // real_ucom_info
```

### 涉及表/SQL

- `real_ucom_info` — 上位机信息表

---

## warning

### 协议命令

```
{ "mkey": "script", "op": "Ucom.warning", "reqid": "xxx", "data": { "senorList": [{ "probe": "...", "value": "..." }] } }
```

### 实现意图

上位机告警信息上报（传感器数据）。直接存储传感器列表 JSON。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.senorList | JSONArray | 是 | 传感器告警列表 |

### 调用链

```
trans.controller.req.script.Ucom.warning(Session, RequestContext)
  → iUcomInfoService.addSumWarning(senorList.toString())   // real_ucom_warning
```

### 涉及表/SQL

- `real_ucom_warning` — 上位机告警表

---

## probe

### 协议命令

```
{ "mkey": "script", "op": "Ucom.probe", "reqid": "xxx", "data": { "probeList": ["probe1", "probe2"] } }
```

### 实现意图

获取上位机探头配置的告警信息。先查全局告警汇总，再按 probeList 过滤匹配的探头，返回匹配项的告警数据。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.probeList | JSONArray | 是 | 探头名称列表 |

### 调用链

```
trans.controller.req.script.Ucom.probe(Session, RequestContext)
  → iUcomInfoService.getWarningByCaseNum("sumWarning")     // real_ucom_warning
  → JSON 过滤：probeList ∩ sumWarning
  → iUcomInfoService.addWarning(ucomid, filteredList)      // real_ucom_warning
```

### 涉及表/SQL

- `real_ucom_warning` — 上位机告警表
