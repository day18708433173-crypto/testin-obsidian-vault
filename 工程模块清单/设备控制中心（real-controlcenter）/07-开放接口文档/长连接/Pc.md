---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# Pc

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.Pc`

上位机配置加载和上报。

## loadCfg

### 协议命令

```
{ "mkey": "script", "op": "Pc.loadCfg", "reqid": "xxx", "data": {} }
```

### 实现意图

上位机启动时加载自身配置信息（RealcfgPcCfg），包括 case 位置、IP、端口等。

### 请求消息字段

无需 data 字段，依靠 sessionKey（即 ucomid）定位配置。

### 响应

```json
{
  "code": 0,
  "data": { /* RealcfgPcCfg JSON */ }
}
```

### 调用链

```
trans.controller.req.script.Pc.loadCfg(Session, RequestContext)
  → ipcconfigservice.get(ucomid)     // [real-cfg](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md) realcfg_pc_cfg
```

### 涉及表/SQL

- `realcfg_pc_cfg` — 上位机配置表

---

## report

### 协议命令

```
{ "mkey": "script", "op": "Pc.report", "reqid": "xxx", "data": { "remoteUrl": "http://...", "ip": "10.0.0.1" } }
```

### 实现意图

上位机上报自身 remoteUrl（真机地址）和 IP 信息。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.remoteUrl | String | 是 | 真机远程访问地址 |
| data.ip | String | 否 | 上位机 IP |

### 调用链

```
trans.controller.req.script.Pc.report(Session, RequestContext)
  → ipcconfigservice.report(ucomid, ip, remoteUrl)    // [real-cfg](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md) realcfg_pc_cfg
```

### 涉及表/SQL

- `realcfg_pc_cfg` — 上位机配置表
