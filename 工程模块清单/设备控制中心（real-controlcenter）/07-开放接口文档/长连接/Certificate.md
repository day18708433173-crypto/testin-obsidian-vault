---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# Certificate

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.Certificate`

iOS 设备证书信息查询。

## list

### 协议命令

```
{ "mkey": "script", "op": "Certificate.list", "reqid": "xxx", "data": { "devices": ["deviceid1", "deviceid2"] } }
```

### 实现意图

根据设备 ID 列表批量查询 iOS 设备的证书信息。先校验设备是否为 iOS（systemName = "ios"），再查询证书列表。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.devices | JSONArray | 是 | 设备 ID 字符串数组 |

### 响应

```json
{
  "code": 0,
  "data": {
    "deviceid1": [{ /* CertificateInfo JSON */ }],
    "deviceid2": [...]
  }
}
```

返回格式：以 deviceid 为 key，证书列表为 value 的 map。

### 调用链

```
trans.controller.req.script.Certificate.list(Session, RequestContext)
  → DevicePoolUtil.getDevicePool().get(deviceid)          // 内存检查设备系统类型
  → icertificateinfoservice.listByDeviceid(deviceid)      // real_device_certificate
```

### 涉及表/SQL

- `real_device_certificate` — 设备证书表

### 异常处理

- devices 为空 → `paraInvalid`
- 设备非 iOS → `paraInvalid`，提示 deviceid is invalid
