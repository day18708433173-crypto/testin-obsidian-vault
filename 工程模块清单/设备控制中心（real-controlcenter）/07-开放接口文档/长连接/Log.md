---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# Log

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.Log`

设备日志信息上报。

## infoReport

### 协议命令

```
{ "mkey": "script", "op": "Log.infoReport", "reqid": "xxx", "data": { "deviceid": "...", "taskid": "...", "subtaskid": "...", "sub_subtaskid": "...", "errCode": 0, "errMsg": "...", "descr": "...", "interrupt": 0 } }
```

### 实现意图

上位机上报设备执行日志（错误信息、描述、中断标志等）。deviceid 字段不做强制校验（注释掉了），允许为 null。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.taskid | String | 否 | 任务 ID |
| data.deviceid | String | 否 | 设备 ID（可选） |
| data.subtaskid | String | 否 | 子任务 ID |
| data.sub_subtaskid | String | 否 | 子子任务 ID |
| data.errCode | int | 是 | 错误编码 |
| data.errMsg | String | 否 | 错误信息 |
| data.descr | String | 否 | 描述信息 |
| data.interrupt | int | 否 | 任务中断标志（0=不中断，1=中断） |

### 响应

```json
{ "code": 0, "data": { "result": 1 } }
```

### 调用链

```
trans.controller.req.script.Log.infoReport(Session, RequestContext)
  → idevicelogservice.report(taskid, DeviceLogInfo)   // real_device_log
```

### 涉及表/SQL

- `real_device_log` — 设备日志表
