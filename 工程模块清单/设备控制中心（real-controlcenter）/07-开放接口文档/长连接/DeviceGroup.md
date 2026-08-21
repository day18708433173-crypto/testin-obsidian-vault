---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# DeviceGroup

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.DeviceGroup`

设备组信息查询。

## get

### 协议命令

```
{ "mkey": "script", "op": "DeviceGroup.get", "reqid": "xxx", "data": { "deviceid": "deviceGroupId" } }
```

### 实现意图

根据设备组 ID 查询设备组详细信息，包含主控设备列表、功能设备列表、手机设备列表等。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.deviceid | String | 是 | 设备组 ID |

### 响应

```json
{
  "code": 0,
  "data": {
    "result": { /* DeviceGroupment JSON，含 mainControlDeviceList, funDeviceList, phoneDeviceList 等 */ }
  }
}
```

### 调用链

```
trans.controller.req.script.DeviceGroup.get(Session, RequestContext)
  → DeviceGroupDAO.get(deviceGroupId)    // real_device_group
```

### 涉及表/SQL

- `real_device_group` — 设备组表
