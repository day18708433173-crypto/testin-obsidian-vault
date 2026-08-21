# service-NoticeReportApi — 跨端任务中心状态上报

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/cross/NoticeReportApi.java`
> 类型：远端代理（→ RealCross 服务）
> 转发方式：V1 ApiServlet 前缀（`ApiUtil.doPress("RealCross", reqJson)`，action/op 反射路由）

## 方法列表

### 1. reportStatus — 报告多端任务中心通知

```java
public Boolean reportStatus(String action, JSONObject contentJson, Integer execStatus) throws GeneralException
```

**用途**：任务开始/结束时向 RealCross 多端任务中心上报执行状态，跨端列表页据此刷新任务状态。

**转发目标**：`action=task, op=TaskNotice.reportStatus`（RealCross）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| action | String | 是 | 任务动作（如 "finish"），null 抛 `paraInvalid` |
| contentJson | JSONObject | 是 | 任务内容（取 crossTaskid/taskid/endTime/result），null 抛 `paraInvalid` |
| execStatus | Integer | 否 | 执行状态（调用处将 3 映射为 2） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Boolean | 远端 result 为 true 时返回 true，否则记 debugLog 返回 false |

**调用者**：
- `TaskProcessServiceImpl.java`（`noticereportapi.reportStatus("finish", contentJson, execStatus)`）
- 注入于 `AbstractGenericServiceImpl`（`noticereportapi` 字段）

## 相关文档

- [00-分支索引](00-分支索引.md)
- RealCross 服务
- [service-CallbackApi](service-CallbackApi.md)
