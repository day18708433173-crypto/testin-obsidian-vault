---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# Device

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.Device`

设备相关的全部长连接命令，包括设备列表查询、设备上报、机型匹配、设备标记、设备占用/释放、串口配置查询。

## infolist

### 协议命令

```
{ "mkey": "script", "op": "Device.infolist", "reqid": "xxx", "data": { "page": 1, "pageSize": 10, ...conditionKeys } }
```

### 实现意图

上位机根据条件查询设备列表。支持通过 `DeviceConditionKeyword` 枚举定义的多种过滤条件（如品牌、型号、系统版本等），分页返回 `ViewDeviceInfo` 列表。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.page | int | 否 | 页码，默认 0 |
| data.pageSize | int | 否 | 每页条数，默认 10 |
| data.* | * | 否 | `DeviceConditionKeyword` 枚举中的条件键，支持多值数组 |

### 响应

```json
{
  "resid": "xxx",
  "mkey": "script",
  "op": "Device.infolist",
  "code": 0,
  "data": { "list": [ /* ViewDeviceInfo 列表 */ ] }
}
```

### 调用链

```
trans.controller.req.script.Device.infolist(Session, RequestContext)
  → iviewdeviceinfodao.baselist(conditionMap, null, page, pageSize)   // view_device_info
```

### 涉及表/SQL

- `view_device_info` 视图（或对应基础表）

---

## report

### 协议命令

```
{ "mkey": "script", "op": "Device.report", "reqid": "xxx", "data": { "devices": [{...}] } }
```

### 实现意图

上位机首次启动时上报名下所有设备的基础信息（品牌、型号、系统版本、序列号等）。缺失关键字段的设备记录为异常设备。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.devices | JSONArray | 是 | 设备信息列表（每项含 deviceid, deviceBrandName, deviceModelName, releaseVer, status 等） |

### 响应

```json
{
  "code": 0,
  "data": { "result": 1, "devices": [ /* DeviceInfoResultDTO 屏幕模式列表 */ ] }
}
```

### 调用链

```
trans.controller.req.script.Device.report(Session, RequestContext)
  → deviceService.getDeviceScreenModeListByUcomid(ucomid)   // 查询屏幕模式
  → ideviceservice.infoReport(DeviceInfo)                    // real_device_info
  → ideviceservice.abnormalReport(ucomid, abnormalDevices)  // 异常设备上报
```

### 涉及表/SQL

- `real_device_info` — 设备信息表

---

## modelinfo

### 协议命令

```
{ "mkey": "script", "op": "Device.modelinfo", "reqid": "xxx", "data": { "deviceid": "xxx" } }
```

### 实现意图

查询设备的机型匹配信息，从内存池获取设备后通过 modelid 查机型详情（品牌、分辨率、OS 等）。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.deviceid | String | 是 | 设备 ID |

### 响应

```json
{
  "code": 0,
  "data": { "brandId": 1, "brandName": "Samsung", "modelid": 100, "modelName": "S21", "dpiWidth": 1080, "dpiHeight": 1920, ... }
}
```

### 调用链

```
trans.controller.req.script.Device.modelinfo(Session, RequestContext)
  → DevicePoolUtil.getDevicePool().get(deviceid)     // 内存设备池
  → imodelservice.get(modelid)                        // [real-cfg](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md) realcfg_model
```

---

## matchSign

### 协议命令

```
{ "mkey": "script", "op": "Device.matchSign", "reqid": "xxx", "data": { "baseImei": "...", "baseAndroidid": "...", "baseMac": "..." } }
```

### 实现意图

通过设备指纹（IMEI/AndroidId/MAC）查询匹配的设备标记信息。至少提供一个标识即可。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.baseImei | String | 否 | 设备 IMEI |
| data.baseAndroidid | String | 否 | Android ID |
| data.baseMac | String | 否 | MAC 地址 |

### 调用链

```
trans.controller.req.script.Device.matchSign(Session, RequestContext)
  → idevicesignservice.match(baseImei, baseAndroidid, baseMac)   // real_device_sign
```

### 涉及表/SQL

- `real_device_sign` — 设备标记表

---

## signReport

### 协议命令

```
{ "mkey": "script", "op": "Device.signReport", "reqid": "xxx", "data": { "deviceid": "xxx", "baseImei": "...", "baseAndroidid": "...", "baseMac": "...", "baseSerialNumber": "..." } }
```

### 实现意图

上位机上报设备的硬件标识信息，用于设备指纹识别和防盗。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.deviceid | String | 是 | 设备 ID |
| data.baseImei | String | 否 | IMEI |
| data.baseAndroidid | String | 否 | Android ID |
| data.baseMac | String | 否 | MAC 地址 |
| data.baseSerialNumber | String | 否 | 序列号 |

### 调用链

```
trans.controller.req.script.Device.signReport(Session, RequestContext)
  → idevicesignservice.report(DeviceSign)   // real_device_sign
```

### 涉及表/SQL

- `real_device_sign` — 设备标记表

---

## occupy

### 协议命令

```
{ "mkey": "script", "op": "Device.occupy", "reqid": "xxx", "data": { "eid": 1, "projectid": 10, "deviceAction": 1, "jobId": "xxx", "deviceid": "xxx", "debugOwner": "user@testin.cn", "type": 0, "markName1": "...", "markName2": "...", "browserType": "...", "browserVer": "..." } }
```

### 实现意图

上位机通知控制中心：某设备已被用户占用。支持三种设备类型，分别创建不同的 Process 记录。PC Client 和 Web 浏览器互斥占用同台上位机。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.type | int | 否 | 设备类型：0=App设备, 1=Web浏览器, 2=PC Client |
| data.eid | int | 是 | 企业 ID |
| data.projectid | int | 是 | 项目组 ID |
| data.deviceAction | int | 是 | 操作类型 |
| data.jobId | String | 是 | 占用任务唯一标识 |
| data.deviceid | String | 是 | 设备 ID（或浏览器/Client 的设备标识） |
| data.debugOwner | String | 是 | 占用者 email |
| data.markName1 | String | 否 | 标记 1 |
| data.markName2 | String | 否 | 标记 2 |
| data.browserType | String | Web时需要 | 浏览器类型 |
| data.browserVer | String | Web时需要 | 浏览器版本 |

### 调用链

```
trans.controller.req.script.Device.occupy(Session, RequestContext)
  // type=DEVICE_TYPE
  → iviewdeviceinfodao.baselist(...)                    // view_device_info
  → userapi.getByEmail(debugOwner)                      // [user-manager](../../../平台基础功能服务/00-首页.md)
  → ideviceprocessservice.occupy(DeviceProcess)         // real_device_process
  // type=WEB_TYPE
  → ipcservice.getOriginalPc(ucomid)                    // real_pc_info / PcInfoPool
  → userapi.getByEmail(debugOwner)
  → iBrowserProcessService.occupy(BrowserProcess)       // real_browser_process
  // type=CLIENT_TYPE
  → iclientservice.getOriginalPc(ucomid)                // real_client_info / ClientInfoPool
  → userapi.getByEmail(debugOwner)
  → iClientProcessService.occupy(ClientProcess)        // real_client_process
```

### 涉及表/SQL

- `real_device_process` — 设备占用记录
- `real_browser_process` — 浏览器占用记录
- `real_client_process` — Client 占用记录
- `view_device_info` — 设备信息视图
- [user-manager](../../../平台基础功能服务/00-首页.md) — 用户查询

---

## release

### 协议命令

```
{ "mkey": "script", "op": "Device.release", "reqid": "xxx", "data": { "jobId": "xxx", "totaltime": 120000, "content": {...}, "type": 0 } }
```

### 实现意图

上位机通知控制中心：设备已被用户释放。记录占用时长。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.type | int | 否 | 设备类型 |
| data.jobId | String | 是 | 占用任务标识 |
| data.totaltime | long | 是 | 占用时长（毫秒） |
| data.content | JSONObject | 否 | 附加内容 |

### 调用链

```
trans.controller.req.script.Device.release(Session, RequestContext)
  // type=DEVICE_TYPE
  → ideviceprocessservice.release(jobId, totaltime, null, content)       // real_device_process
  // type=WEB_TYPE
  → iBrowserProcessService.release(jobId, totaltime, null, content)      // real_browser_process
  // type=CLIENT_TYPE
  → iClientProcessService.release(jobId, totaltime, null, content)       // real_client_process
```

---

## getSerialConfig

### 协议命令

```
{ "mkey": "script", "op": "Device.getSerialConfig", "reqid": "xxx", "data": { "deviceid": "xxx" } }
```

### 实现意图

查询设备的串口配置信息。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.deviceid | String | 是 | 设备 ID |

### 调用链

```
trans.controller.req.script.Device.getSerialConfig(Session, RequestContext)
  → upgradeLogService.getSerialByDeviceId(deviceid)
```
