---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# HarmonyPc

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.HarmonyPc`

鸿蒙 PC 设备信息上报。

## report

### 协议命令

```
{ "mkey": "script", "op": "HarmonyPc.report", "reqid": "xxx", "data": { "harmonyPcs": [{ "pcId": "...", "systemType": "Mac", "systemVersion": "10.16", "systemName": "Mac OS X", "systemBitness": "64", "cpuName": "Apple M1", "cpuArch": "x86_64", "ram": "16.00 G", "ip": "10.10.210.61", "brandName": "Apple Inc.", "protocol": "vnc" }] } }
```

### 实现意图

上报鸿蒙 PC 设备信息。注意：请求的 sessionKey 是鸿蒙 PC **所在上位机**的 ID，而非鸿蒙 PC 自身的 ID。鸿蒙 PC 的 ucomid 由 `pcId + "@testin.cn"` 拼接而成。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.harmonyPcs | JSONArray | 是 | 鸿蒙 PC 列表，每项同 ClientInfo 格式 |

### 调用链

```
trans.controller.req.script.HarmonyPc.report(Session, RequestContext)
  → ClientInfoPojo.parseJson(clientInfoJsonObj)        // 解析
  → 设置 ucomid = pcId + "@testin.cn"
  → 设置 fromUcomDevice = session的ucomid
  → ClientInfoPoolUtil.getClientPool().get(ucomid)      // 内存池
  → iclientservice.report(clientInfoPojo)               // [real-cfg](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md)
```

### 涉及表/SQL

- `real_client_info` — PC 客户端信息表（鸿蒙 PC 也写入此表）

### 关键代码摘录

```java
// 鸿蒙 PC ucomid 构造
clientInfoPojo.setUcomid(clientInfoPojo.getPcId() + "@testin.cn");
clientInfoPojo.setFromUcomDevice(ucomid);  // fromUcomDevice = 上位机ID
```
