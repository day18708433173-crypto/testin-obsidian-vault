# service-TestPlanV3Api — 测试计划 V3 接口

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/plan/TestPlanV3Api.java`（@Service）
> 类型：V3 REST 客户端 → [RealLogfile](../../../平台基础功能服务/00-首页.md)
> 基础 URL：`Config.LOG_FILE_URL`

## 方法列表

### 1. resetTask — 重设任务

```java
public Integer resetTask(ReSetTaskRequestDTO requestDTO)
```
- `POST /v3/test_plan/execute_record_tasks/reset_task`

**请求参数**（`ReSetTaskRequestDTO`，整体必填）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskId | String | 是 | 任务 id |
| taskName | String | 否 | 任务名 |
| suiteId | Integer | 否 | 套件 id |
| scriptTotal | Integer | 否 | 脚本总数 |
| deviceTotal | Integer | 否 | 设备总数 |
| content | String | 否 | 任务内容 |
| executeRecordTaskId | Long | 否 | 执行记录任务 id |
| deviceIds | List | 否 | 设备 id 列表 |
| scriptNos | List | 否 | 脚本号列表 |
| userId | Integer | 否 | 用户 id |
| deviceResponse | Object | 否 | 设备响应 |
| resetStrategy | Object | 否 | 重测策略 |
| enablePreTask / enablePostTask | Boolean | 否 | 前置/后置任务开关 |

> 子字段必填性由 RealLogfile 服务校验，代码未确认。

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Integer | 重设结果（`ResultResponseDTO<Integer>.result`） |

### 2. recordRelation — 执行记录关联

```java
public List<ExecuteTaskWithExecuteRecordResponseDTO> recordRelation(List<String> executeTaskIds)
```
- `POST /v3/test_plan/execute_record_tasks/relations`

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| executeTaskIds | List&lt;String&gt; | 否 | 执行任务 id 列表（无 null 校验） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | ExecuteTaskWithExecuteRecordResponseDTO | executeTaskId → executeRecordId/executeRecordName/planInfoName 映射 |

### 3. taskInfo — 更新任务信息

```java
public Integer taskInfo(Integer taskId, TaskInfoRequestDTO taskInfoDTO)
```
- `PUT /v3/test_plan/task_infos/{taskId}`

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskId | Integer | 是 | 任务 id（URL 路径） |
| taskInfoDTO | TaskInfoRequestDTO | 否 | 更新信息，null 返回 0 |

`TaskInfoRequestDTO` 字段：projectId, userId, taskName, taskType, suiteId, deviceTotal, scriptTotal, needUpdateScript, scriptRowInfo, needUpdateDevice, deviceResponse（必填性由 RealLogfile 校验，代码未确认）。

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Integer | 更新结果 |

### 4. taskRelation — 查询任务关联的计划

```java
public List<RelationResponseDTO> taskRelation(List<Integer> taskIds)
```
- `POST /v3/test_plan/plan_tasks/relations`

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskIds | List&lt;Integer&gt; | 否 | 任务 id 列表，空返回空列表 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | RelationResponseDTO | taskId → planInfos（id, planInfoName, planDeviceStatus）映射 |

### 5. getTestPlanById — 获取测试计划

```java
public PlanInfoResponseDTO getTestPlanById(Long testPlanId)
```
- `GET /v3/test_plan/test_plans/{id}`

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| testPlanId | Long | 否 | 测试计划 id，null 返回 null |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | PlanInfoResponseDTO | 测试计划信息对象（字段见 RealLogfile 服务，代码未确认） |

### 6. scriptFails — 脚本失败详情

```java
public ScriptFailResponseDTO scriptFails(String executeTaskId)
```
- `GET /v3/test_plan/execute_records/report/script_fails?execute_task_id=...`

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| executeTaskId | String | 是 | 执行任务 id |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | ScriptFailResponseDTO | 失败脚本信息对象（字段见 RealLogfile 服务，代码未确认） |

### 7. listScriptExecuteReport — 脚本执行报告

```java
public List<ScriptExecuteReportResponseDTO> listScriptExecuteReport(ScriptExecuteReportRequestDTO request)
```
- `POST /v3/test_plan/execute_records/script_report`（分页，每页500，循环全部页）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| request | ScriptExecuteReportRequestDTO | 是 | 查询条件对象（代码直接 `setPage`，null 会 NPE） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | ScriptExecuteReportResponseDTO | 脚本执行报告对象（字段见 RealLogfile 服务，代码未确认） |

## 被调用方

| 调用者 | 方法 | 场景 |
|--------|------|------|
| TemplateService.listByCondition | taskRelation | 模板列表查询关联状态 |
| TemplateService.batchRemove | taskRelation | 批量删除前检查关联 |
| TemplateService.copyTemplate | — | 关联检查 |
| TestPlanExcelService.reportExcel | getTestPlanById | Excel导出时获取计划名 |
| GenerateReportServiceImpl.generatePlanExcel | scriptFails | Plan Excel生成 |
| NewTaskService.scriptResetTask | resetTask | 脚本重测 |
| BaseQuartz.checkTestPlan | — | 模板删除前检查 |

## 外部服务

| 服务 | 前缀 | 接口 |
|------|------|------|
| [RealLogfile](../../../平台基础功能服务/00-首页.md) | `/v3/test_plan/*` | execute_record_tasks, plan_tasks, test_plans, execute_records |

## 相关文档

- [00-分支索引](00-分支索引.md)
- [service-PlanRecordApi](service-PlanRecordApi.md)
- [service-PlanRecordTaskApi](service-PlanRecordTaskApi.md)
