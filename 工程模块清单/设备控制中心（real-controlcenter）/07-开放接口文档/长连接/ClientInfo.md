---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# ClientInfo

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.ClientInfo`

PC 客户端信息上报。

## report

### 协议命令

```
{ "mkey": "script", "op": "ClientInfo.report", "reqid": "xxx", "data": { "pc": { "pcId": "...", "systemType": "windows", "systemVersion": "10.0.19042", "systemName": "DESKTOP-XXX", "systemBitness": "64", "cpuName": "Intel...", "cpuArch": "amd64", "ram": "8G", "ip": "10.32.20.228", "brandName": "Acer" } } }
```

### 实现意图

PC 客户端启动时上报自身硬件和系统信息。如果 status 为空，从内存池中获取上一次的状态继承。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.pc.pcId | String | — | PC 唯一标识 |
| data.pc.systemType | String | — | 系统类型（windows/mac） |
| data.pc.systemVersion | String | — | 系统版本 |
| data.pc.systemName | String | — | 计算机名 |
| data.pc.systemBitness | String | — | 系统位数 |
| data.pc.cpuName | String | — | CPU 名称 |
| data.pc.cpuArch | String | — | CPU 架构 |
| data.pc.ram | String | — | 内存大小 |
| data.pc.ip | String | — | IP 地址 |
| data.pc.brandName | String | — | 品牌名 |

### 响应

```json
{ "code": 0, "data": { "result": 1 } }
```

### 调用链

```
trans.controller.req.script.ClientInfo.report(Session, RequestContext)
  → ClientInfoPojo.parseJson(reqJson)                          // 解析
  → ClientInfoPoolUtil.getClientPool().get(ucomid)              // 内存池获取状态
  → iclientservice.report(clientInfoPojo)                       // 写入 [real-cfg](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md)
```

### 涉及表/SQL

- `real_client_info` — PC 客户端信息表

### 关键代码摘录

```java
// 处理无状态问题 — 继承上一次的状态
if (clientInfoPojo.getStatus() == null) {
    ClientInfoPojo poolClientInfoPojo = ClientInfoPoolUtil.getClientPool().get(ucomid);
    if (poolClientInfoPojo != null) {
        clientInfoPojo.setStatus(poolClientInfoPojo.getStatus());
    }
}
```
