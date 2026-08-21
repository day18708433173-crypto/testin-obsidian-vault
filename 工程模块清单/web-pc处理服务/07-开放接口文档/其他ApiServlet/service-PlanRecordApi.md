# service-PlanRecordApi — 测试计划执行记录代理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/plan/PlanRecordApi.java`（继承 `AbstractApi`）
> 类型：远端代理（→ RealLogfile 服务）
> 转发方式：HttpClient 直连（非 RestTemplate）；域名从 `service.api.properties` 读 `RealLogfile`；请求头带 `timestamp`

## 方法列表

### 1. getExecuteRecord — 按 id 获取执行记录

```java
public ExecuteRecordResponseDTO getExecuteRecord(Long executeRecordId)
```

**转发目标**：`GET {RealLogfile}/v3/test_plan/execute_records?id={executeRecordId}`；取返回列表第一条。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| executeRecordId | Long | 是 | 执行记录 id |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | ExecuteRecordResponseDTO | 执行记录对象（字段见 RealLogfile 服务，代码未确认） |

**调用者**：`TestPlanExcelService.java` — 导出计划执行 Excel。

### 2. updateRecord — 回写执行记录 Excel 下载地址

```java
public Boolean updateRecord(Long executeRecordId, String downloadUrl, String executeRecordName)
```

**转发目标**：`PUT {RealLogfile}/v3/test_plan/execute_records/{executeRecordId}`，body JSON 含 `executeRecordName/reportExcelUrl`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| executeRecordId | Long | 是 | 执行记录 id（拼接到 URL 路径） |
| downloadUrl | String | 否 | Excel 下载地址 |
| executeRecordName | String | 否 | 执行记录名称 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Boolean | 更新成功为 true，失败/异常为 false |

**调用者**：`GenerateReportServiceImpl.java` — 报告生成后回写 excelUrl。

### 3. getExecuteRecordByTaskId — 按任务 id 获取执行记录

```java
public ExecuteRecordResponseDTO getExecuteRecordByTaskId(String executeTaskId)
```

**转发目标**：`GET {RealLogfile}/v3/test_plan/execute_records/report?execute_task_id={executeTaskId}`；按 `code==0` 解析 `data`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| executeTaskId | String | 是 | 执行任务 id |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | ExecuteRecordResponseDTO | 执行记录对象（字段见 RealLogfile 服务，代码未确认） |

**调用者**：`GenerateReportServiceImpl.java`。

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealLogfile](../../../平台基础功能服务/00-首页.md)
- [service-PlanRecordTaskApi](service-PlanRecordTaskApi.md)
