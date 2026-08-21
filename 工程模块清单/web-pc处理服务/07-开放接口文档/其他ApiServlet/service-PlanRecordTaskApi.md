# service-PlanRecordTaskApi — 执行记录子任务查询代理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/plan/PlanRecordTaskApi.java`（继承 `AbstractApi`）
> 类型：远端代理（→ RealLogfile 服务）
> 转发方式：HttpClient 直连；域名从 `service.api.properties` 读 `RealLogfile`；请求头带 `timestamp`

## 方法列表

### 1. getExecuteTaskRecord — 获取执行记录下的子任务列表

```java
public List<ExecuteRecordTaskResponseDTO> getExecuteTaskRecord(Long executeRecordId)
```

**用途**：按执行记录 id 查询其下所有执行任务（一次计划执行拆出的多个 execute_task）。

**转发目标**：

```java
HttpGet httpGet = new HttpGet(actionUrl + "/v3/test_plan/execute_record_tasks?execute_record_id=" + executeRecordId);
```

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| executeRecordId | Long | 是 | 执行记录 id |

**返回参数**：`List<ExecuteRecordTaskResponseDTO>`（gson 解析 `jsonList`）；异常或无数据返回 null。

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | ExecuteRecordTaskResponseDTO | 执行子任务对象（字段见 RealLogfile 服务，代码未确认） |

**调用者**：`GenerateReportServiceImpl.java` — 生成报告时先取子任务，再用子任务的 executeTaskId 调 [service-PlanRecordApi](service-PlanRecordApi.md) 的 `getExecuteRecordByTaskId`。

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealLogfile](../../../平台基础功能服务/00-首页.md)
- [service-PlanRecordApi](service-PlanRecordApi.md)
