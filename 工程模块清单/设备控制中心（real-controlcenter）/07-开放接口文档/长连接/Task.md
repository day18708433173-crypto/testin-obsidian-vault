---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# Task

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.Task`

任务生命周期管理，包括任务预完成、任务完成、结果上报、过程上报、视频上报、执行申请、抓包上报、安全变量密钥获取。

## precomplete

### 协议命令

```
{ "mkey": "script", "op": "Task.precomplete", "reqid": "xxx", "data": { "deviceid": "...", "taskid": "...", "subtaskid": "...", "subsubtaskid": "...", "delayperiod": 0, "runinfo": {...}, "inputParams": [...], "params": [...] } }
```

### 实现意图

任务预完成上报。上位机执行完某个子任务后，上报运行信息、输入参数、输出参数，并可设置延迟下次任务匹配的时间（delayperiod）。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.deviceid | String | 是 | 设备 ID |
| data.taskid | String | 否 | 任务 ID |
| data.subtaskid | String | 是 | 子任务 ID |
| data.subsubtaskid | String | 是 | 子子任务 ID |
| data.delayperiod | long | 否 | 推迟下次匹配时间（毫秒） |
| data.runinfo | JSONObject | 否 | 运行结果信息 |
| data.inputParams | JSONArray | 否 | 输入参数 |
| data.params | JSONArray | 否 | 输出参数 |

### 调用链

```
trans.controller.req.script.Task.precomplete(Session, RequestContext)
  → itaskinfoservice.precomplete(taskid, subtaskid, subsubtaskid, deviceid, delayperiod, runinfo, inputParams, params, ucomid)
    → [real-scheduling](../../../任务调度服务（real-scheduling）/07-开放接口文档/00-模块索引.md)
```

---

## complete

### 协议命令

```
{ "mkey": "script", "op": "Task.complete", "reqid": "xxx", "data": { "deviceid": "...", "subtaskid": "...", "deviceType": 0 } }
```

### 实现意图

任务完成通知，异常释放设备。根据 deviceType 分别调用对应的 ProcessService 做 releaseByAbnormal。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.deviceid | String | 是 | 设备 ID |
| data.subtaskid | String | 是 | 子任务 ID |
| data.deviceType | int | 是 | 设备类型（App/Web/Pc） |

### 调用链

```
trans.controller.req.script.Task.complete(Session, RequestContext)
  // App
  → IDeviceProcessService.releaseByAbnormal(deviceid, "complete")   // real_device_process
  // Web
  → IBrowserProcessService.releaseByAbnormal(ucomid, "complete")    // real_browser_process
  // PC
  → IClientProcessService.releaseByAbnormal(ucomid, "complete")     // real_client_process
```

---

## resultReport

### 协议命令

```
{ "mkey": "script", "op": "Task.resultReport", "reqid": "xxx", "data": { "deviceid": "...", "taskid": "...", "subtaskid": "...", "subsubtaskid": "...", "resultUrl": "http://...", "retryNum": 0, "standard": {...}, "sid": "..." } }
```

### 实现意图

任务执行结果上报，提交测试结果 URL。支持重试序号（retryNum）。Web 任务还需 standard（执行策略）和 sid（session）。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.deviceid | String | 是 | 设备 ID |
| data.taskid | String | 否 | 任务 ID |
| data.subtaskid | String | 是 | 子任务 ID |
| data.subsubtaskid | String | 是 | 子子任务 ID |
| data.resultUrl | String | 是 | 测试结果地址 |
| data.retryNum | int | 是 | 重试序号 |
| data.standard | JSONObject | 否 | Web 执行策略 |
| data.sid | String | 否 | 用户 session ID |

### 调用链

```
trans.controller.req.script.Task.resultReport(Session, RequestContext)
  → itaskinfoservice.reportResult(taskid, subtaskid, subsubtaskid, deviceid, retryNum, resultUrl, standard, sid)
    → [real-scheduling](../../../任务调度服务（real-scheduling）/07-开放接口文档/00-模块索引.md)
```

---

## processReport

### 协议命令

```
{ "mkey": "script", "op": "Task.processReport", "reqid": "xxx", "data": { "deviceid": "...", "taskid": "...", "subtaskid": "...", "subsubtaskid": "..." } }
```

### 实现意图

任务执行过程上报，通知调度服务任务仍在执行中，防止超时判定。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.deviceid | String | 是 | 设备 ID |
| data.taskid | String | 是 | 任务 ID |
| data.subtaskid | String | 是 | 子任务 ID |
| data.subsubtaskid | String | 是 | 子子任务 ID |

### 调用链

```
trans.controller.req.script.Task.processReport(Session, RequestContext)
  → itaskinfoservice.processReport(taskid, subtaskid, subsubtaskid, deviceid)
    → [real-scheduling](../../../任务调度服务（real-scheduling）/07-开放接口文档/00-模块索引.md)
```

---

## videoReport

### 协议命令

```
{ "mkey": "script", "op": "Task.videoReport", "reqid": "xxx", "data": { "taskid": "...", "subtaskid": "...", "subsubtaskid": "...", "videoUrl": "http://..." } }
```

### 实现意图

任务执行视频上报。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.taskid | String | 是 | 任务 ID |
| data.subtaskid | String | 是 | 子任务 ID |
| data.subsubtaskid | String | 是 | 子子任务 ID |
| data.videoUrl | String | 是 | 视频地址 |

### 调用链

```
trans.controller.req.script.Task.videoReport(Session, RequestContext)
  → itaskinfoservice.reportVideo(taskid, subtaskid, subsubtaskid, videoUrl)
    → [real-scheduling](../../../任务调度服务（real-scheduling）/07-开放接口文档/00-模块索引.md)
```

---

## applyForExecution

### 协议命令

```
{ "mkey": "script", "op": "Task.applyForExecution", "reqid": "xxx", "data": { "taskid": "...", "subtaskid": "...", "subsubtaskid": "...", "deviceid": "..." } }
```

### 实现意图

任务执行前向调度服务申请确认。带有 2 秒的 sleep 等待。调用调度服务的静态方法 `TaskApi.applyForExecution`。

### 调用链

```
trans.controller.req.script.Task.applyForExecution(Session, RequestContext)
  → Thread.sleep(2000)
  → TaskApi.applyForExecution(subtaskid, subsubtaskid, deviceid, taskid)   // [real-scheduling](../../../任务调度服务（real-scheduling）/07-开放接口文档/00-模块索引.md)
```

---

## pcapReport

### 协议命令

```
{ "mkey": "script", "op": "Task.pcapReport", "reqid": "xxx", "data": { "pcapInfo": { "subtaskid": "...", "deviceid": "...", "content": "..." } } }
```

### 实现意图

网络抓包信息上报。解析 pcapInfo 中的 steps 步骤信息，计算起止时间，匹配设备信息和错误码分类，存储抓包记录和 App 信息。

### 调用链

```
trans.controller.req.script.Task.pcapReport(Session, RequestContext)
  → itaskinfoservice.getCoding(stepErrorCode, warningCode)     // 错误码映射
  → iUcomInfoService.addPcapInfo(PcapInfo)                     // real_pcap_info
  → iUcomInfoService.addAppInfo(AppInfo)                        // real_app_info
```

### 涉及表/SQL

- `real_pcap_info` — 抓包信息表
- `real_app_info` — 应用信息表

---

## pcapFileReport

### 协议命令

```
{ "mkey": "script", "op": "Task.pcapFileReport", "reqid": "xxx", "data": { "id": "...", "pcapFileUrl": "...", "imageZipUrl": "..." } }
```

### 实现意图

抓包文件上报（中科院需求）。下载图片压缩包，解压并逐个上传到文件服务，更新 pcapInfo 中的图片 URL。

### 调用链

```
trans.controller.req.script.Task.pcapFileReport(Session, RequestContext)
  → iUcomInfoService.getPcapInfoById(id)          // real_pcap_info
  → FileUtils.saveToFile(imageZipUrl, ...)        // 下载压缩包
  → FileUtils.extractZip(filePath)                // 解压
  → uploadApi.upload(imgPath, ...)                // [file-service](../../../文件管理服务/00-首页.md)
  → iUcomInfoService.updatePcapInfo(PcapInfo)     // real_pcap_info
```

---

## getProjectGlobalVariablesSecretKey

### 协议命令

```
{ "mkey": "script", "op": "Task.getProjectGlobalVariablesSecretKey", "reqid": "xxx", "data": {} }
```

### 实现意图

获取项目全局变量加密的秘钥，用于上位机侧安全变量加解密。

### 调用链

```
trans.controller.req.script.Task.getProjectGlobalVariablesSecretKey(Session, RequestContext)
  → ProjectGlobalVariablesV3Api.getSecretKey()   // [real-test](../../../app处理服务（real-test）/07-开放接口文档/00-模块索引.md)
```
