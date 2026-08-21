# TaskController — 任务管理（提测/取消/删除/查询）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/TaskController.java`
> 类级路由：`/task`
> Service 实现：`cn.testin.business.impl.task.TaskServiceImpl`（本控制器全部委托给 `ITaskService`）
> 业务：测试任务的生命周期管理——提测、取消（批量/单个/按子任务）、删除（单个/批量）、脚本重置、按 jobId/任务id 查询详情、执行完成数统计、任务进度同步。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 | 操作日志 |
|---|---|---|---|---|---|
| 1 | POST | `/v3/task/tasks/cancel` | batchCancelTasks | 任务取消（批量） | `TASK_STOP` |
| 2 | POST | `/v3/task/tasks` | addTask | 任务提测 | 无 |
| 3 | DELETE | `/v3/task/tasks/{task_id}` | deleteTask | 删除单个任务 | `TASK_DELETE` |
| 4 | POST | `/v3/task/tasks/batch_delete` | batchDeleteTasks | 批量删除任务 | `TASK_DELETE` |
| 5 | POST | `/v3/task/tasks/script_reset` | scriptResetTask | 脚本任务重置 | 无 |
| 6 | GET | `/v3/task/tasks/list` | getTasksByJobId | 通过 jobId 查询门户任务列表（分页） | 无 |
| 7 | GET | `/v3/task/execute/finish_count` | getTaskExecuteFinishCount | 任务执行完成数统计 | 无 |
| 8 | POST | `/v3/task/sync/task_process` | syncTaskProcess | 任务进度同步 | 无 |
| 9 | POST | `/v3/task/tasks/task_cancel` | cancelOneTask | 取消单个任务，支持按子任务id取消 | 无 |
| 10 | GET | `/v3/task/tasks/{task_id}` | getTaskDetail | 通过任务id获取详细信息 | 无 |
| 11 | GET | `/v3/task/{job_id}` | getTaskDetail | 通过 jobId + task_type 获取详细信息 | 无 |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`；写操作大多返回 `BaseDataResultDTO { Long result }`（影响行数）。
GET 查询接口带 `@UnderlineToCamel`：下划线 query 参数自动转驼峰绑定 DTO。

---

## 1. POST /v3/task/tasks/cancel — 任务取消（批量）

### 入口

`TaskController.batchCancelTasks(@RequestBody @Valid TaskCancelRequestDTO requestDTO)`

### 请求参数（TaskCancelRequestDTO，JSON Body，@Valid）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskIds | List\<String\> | 是 | 任务id列表，`@NotEmpty(任务id不能为空)` |
| projectId | Integer | 是 | 项目id，`@NotNull(项目id不能为空)` |
| eid | Integer | 否 | 企业id |
| userId / userName | Integer / String | 否 | 操作人 |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 取消成功的任务数。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | BaseDataResultDTO | 结果对象 |
| data.result | Long | 取消成功的任务数 |

### 调用链

```
TaskController.batchCancelTasks
├─ @OperateLog(TASK_STOP) AOP 记录操作日志
└─ TaskServiceImpl.batchCancelTasks
```

---

## 2. POST /v3/task/tasks — 任务提测

### 入口

`TaskController.addTask(@RequestBody @Valid TaskAddRequestDTO taskAddRequest)`

### 请求参数（TaskAddRequestDTO，JSON Body，@Valid）

核心字段（完整字段见 `cn.testin.dto.request.task.TaskAddRequestDTO`）：

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| Body | TaskAddRequestDTO | 是 | 任务信息（JSON Body；字段级必填性代码未确认） |
| eid / projectId / userId | Integer / Integer / Integer | 是 | 企业 / 项目 / 创建人 |
| taskName / taskType / bizCode | String / String / String | 是 | 任务名称 / 任务类型 / 业务编码 |
| executeType | Integer | 否 | 执行方式：1 分布式执行（原快速执行和数据驱动），2 按顺序执行 |
| execStandard | TaskExecStandardDTO | 否 | 执行标准 |
| scripts | List\<TaskScriptsInfoDTO\> | 否 | 脚本数据列表 |
| devices / taskDeviceCondition / checkDevice | Object | 否 | 设备数据与设备筛选条件 |
| dataSource | TaskDataSourceInfoDTO | 否 | 数据源 |
| suiteInfo | Object | 否 | 应用相关数据（taskType 为 App 时有用） |
| networks / simulateNetworkName | Object / String | 否 | 网络相关 / 模拟网络名称 |
| quartzInfo | CronQuartzDTO | 否 | 定时任务信息，配合 jobId |
| taskNotice | TaskNoticeDTO | 否 | 通知配置 |
| taskReleaseTimePeriodsList | List | 否 | 任务执行时间段控制 |
| envId | Integer | 否 | 脚本含 SQL 步骤时连接的环境id |
| callbackUrl / params / additionalInfo | String / Object / String | 否 | 回调地址 / 扩展参数 |
| saveTemplate / dirId | Boolean / Integer | 否 | 是否存为模板 / 模板目录id |
| executeRecordId / executeRecordTaskId / executeRecordTaskName | Long / Long / String | 否 | 关联执行记录 |
| level | Integer | 否 | 执行优先级，越小优先级越高 |
| planInfoId / jobId / onlyReset / resetInfo | Long / Integer / Boolean / Object | 否 | 计划关联 / 定时 jobId / 仅重置 / pmreal 重置信息 |

### 响应结构

`ResponseResult<Map<String,String>>`，`data.result` = 新任务id（字符串）。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | Map\<String, String\> | 结果对象 |
| data.result | String | 新任务id |

### 实现意图

创建并下发一个测试任务（可关联定时 job、计划、执行记录）；Service 方法带 `@Transactional(rollbackFor = Exception.class)`，任务主记录与脚本/设备/数据源明细整体成败一致。

### 调用链

```
TaskController.addTask
└─ TaskServiceImpl.addTask (@Transactional)
```

---

## 3. DELETE /v3/task/tasks/{task_id} — 删除单个任务

### 入口

`TaskController.deleteTask(@PathVariable("task_id") String taskId, BaseQueryRequestDTO request)`（`@UnderlineToCamel`）

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 任务id（路径参数） |
| 其余 | BaseQueryRequestDTO | 否 | 附加条件（Query，下划线自动转驼峰，含操作人等基础字段） |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | BaseDataResultDTO | 结果对象 |
| data.result | Long | 影响行数 |

### 调用链

```
TaskController.deleteTask
├─ @OperateLog(TASK_DELETE)
└─ TaskServiceImpl.deleteTask(taskId, request)
```

---

## 4. POST /v3/task/tasks/batch_delete — 批量删除任务

### 入口

`TaskController.batchDeleteTasks(@RequestBody @Valid TaskBatchDeleteRequestDTO request)`

### 请求参数（TaskBatchDeleteRequestDTO，JSON Body，@Valid）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskIds | List\<String\> | 是 | 任务id列表，`@NotEmpty(任务id不能为空)` |
| projectId | Integer | 是 | 项目id，`@NotNull(项目id不能为空)` |
| eid / userId / userName | Integer / Integer / String | 否 | 企业 / 操作人 |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 删除行数。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | BaseDataResultDTO | 结果对象 |
| data.result | Long | 删除行数 |

### 调用链

```
TaskController.batchDeleteTasks
├─ @OperateLog(TASK_DELETE)
└─ TaskServiceImpl.batchDeleteTasks
```

---

## 5. POST /v3/task/tasks/script_reset — 脚本任务重置

### 入口

`TaskController.scriptResetTask(@RequestBody @Valid ScriptTaskResetRequestDTO request)`

### 请求参数（ScriptTaskResetRequestDTO，JSON Body，继承 BaseRequestDTO）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskId | String | 是 | 任务id（代码未确认） |
| taskType | Integer | 否 | 任务类型 |
| executeRecordTaskId | Long | 否 | 执行记录任务id |
| subSubTaskId | List | 否 | 子子任务id列表 |

### 响应结构

`ResponseResult<BaseResultDTO>`，`data.result` = Service 返回的结果串。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | BaseResultDTO | 结果对象 |
| data.result | String | Service 返回的结果串 |

### 调用链

```
TaskController.scriptResetTask
└─ TaskServiceImpl.scriptResetTask
```

---

## 6. GET /v3/task/tasks/list — 通过 jobId 查询门户任务列表

### 入口

`TaskController.getTasksByJobId(jobId, projectId, bizCode, eid, page, pageSize)`（全部 Query 参数）

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| job_id | Integer | 是 | 定时任务/执行 job id |
| project_id | Integer | 是 | 项目id |
| biz_code | String | 是 | 业务编码 |
| eid | Integer | 是 | 企业id |
| page / page_size | Integer | 是 | 分页参数 |

Controller 将 6 个参数组装为 `TaskQueryRequestDTO` 交 Service。

### 响应结构

`ResponseResult<BasePageListResponseDTO<DbPortalTask>>`：分页门户任务列表。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | Object | 分页结果对象 |
| data.page | Integer | 当前页码 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Integer | 总行数 |
| data.list | List\<DbPortalTask\> | 门户任务列表 |

### 实现意图

查询某 job 下的门户任务记录；返回前在 Controller 层做两类转换：

- 状态整合：`TaskStatusEnum.getNewStatusByOldStatus(bizCode + "_" + taskStatus)` 把各端旧状态映射为统一新状态；
- bizCode 归一：`JUNIT_TEST → NEW_APP_TEST`、`WEB_TEST → NEW_WEB_TEST`、`PC_TEST → NEW_DESKTOP_TEST`（app/web/pc 三端统一）。

### 调用链

```
TaskController.getTasksByJobId
└─ TaskServiceImpl.getTasksByJobId
   （返回后 Controller 层做状态/bizCode 转换）
```

---

## 7. GET /v3/task/execute/finish_count — 任务执行完成数统计

### 入口

`TaskController.getTaskExecuteFinishCount(TaskExecuteCountStatisticRequestDTO requestDTO)`（`@UnderlineToCamel`）

### 请求参数（Query，继承 BaseRequestDTO）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| startTime / endTime | Long | 否 | 执行开始/结束时间（Long 时间戳） |
| taskType | Integer | 否 | 任务类型 |

### 响应结构

`ResponseResult<TaskExecuteFinishDTO>`：完成数统计结果。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | TaskExecuteFinishDTO | 完成数统计结果（字段代码未确认） |

### 调用链

```
TaskController.getTaskExecuteFinishCount
└─ TaskServiceImpl.getTaskExecuteFinishCount
```

---

## 8. POST /v3/task/sync/task_process — 任务进度同步

### 入口

`TaskController.syncTaskProcess()`（无请求体参数）

### 请求参数

无。

### 响应结构

`ResponseResult<BaseDataResultDTO>`（空 result 包装）。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | BaseDataResultDTO | 结果对象（空 result 包装） |

### 实现意图

触发一次任务进度同步（刷新各任务执行进度到门户侧数据）；Service 方法带 `@Transactional(rollbackFor = Exception.class)`。

### 调用链

```
TaskController.syncTaskProcess
└─ TaskServiceImpl.syncTaskProcess (@Transactional)
```

---

## 9. POST /v3/task/tasks/task_cancel — 取消单个任务（支持子任务）

### 入口

`TaskController.cancelOneTask(@RequestBody @Valid OneTaskCancelRequestDTO requestDTO)`

### 请求参数（OneTaskCancelRequestDTO，JSON Body，@Valid）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskId | String | 是 | 任务id，`@NotEmpty(任务id不能为空)` |
| projectId | Integer | 是 | 项目id，`@NotNull(项目id不能为空)` |
| taskType | Integer | 否 | 任务类型 |
| subTaskId | String | 否 | 子任务id；传了则只取消该子任务 |
| eid / userId / userName | Integer / Integer / String | 否 | 企业 / 操作人 |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | BaseDataResultDTO | 结果对象 |
| data.result | Long | 影响行数 |

### 调用链

```
TaskController.cancelOneTask
└─ TaskServiceImpl.cancelOneTask
```

---

## 10. GET /v3/task/tasks/{task_id} — 通过任务id获取详细信息

### 入口

`TaskController.getTaskDetail(@PathVariable("task_id") String taskId, @RequestParam(value="script_status", required=false) String scriptStatus)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 任务id（路径参数） |
| script_status | String | 否 | 按脚本状态过滤任务内脚本明细（Query） |

### 响应结构

`ResponseResult<TaskInfoResponseDTO>`：任务详情。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | TaskInfoResponseDTO | 任务详情（字段代码未确认） |

### 调用链

```
TaskController.getTaskDetail
└─ TaskServiceImpl.getTaskDetailById(taskId, scriptStatus)
```

---

## 11. GET /v3/task/{job_id} — 通过 jobId 获取详细信息

### 入口

`TaskController.getTaskDetail(@PathVariable("job_id") Integer jobId, @RequestParam("task_type") Integer taskType)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| job_id | Integer | 是 | 定时/执行 job id（路径参数） |
| task_type | Integer | 是 | 任务类型（Query） |

### 响应结构

`ResponseResult<TaskInfoResponseDTO>`：任务详情。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | TaskInfoResponseDTO | 任务详情（字段代码未确认） |

### 调用链

```
TaskController.getTaskDetail
└─ TaskServiceImpl.getTaskDetailByJobId(jobId, taskType)
```

---

## 备注

- 本 Controller 为纯转发层：参数由 `@Valid` 注解兜底（taskIds/projectId/taskId 非空），业务在 `TaskServiceImpl`。
- 事务方法（已核实 `@Transactional(rollbackFor = Exception.class)`）：`addTask`、`syncTaskProcess`。
- 注意两个重载的 `getTaskDetail`：`/v3/task/tasks/{task_id}`（String taskId）与 `/v3/task/{job_id}`（Integer jobId），路径前缀不同不会冲突，但注意不要漏写 `tasks/` 段。
- `getTasksByJobId` 的状态/bizCode 转换逻辑与旧门户接口 `cn.testin.service.portal.Task.list` 中的 v3 转换逻辑一致（见 [portal-Task](portal-Task.md)）。
- 请求 DTO 统一在 `cn.testin.dto.request.task`（`BaseQueryRequestDTO` 在 `cn.testin.dto.request`），响应 DTO 在 `cn.testin.dto.response.task`。

相关文档：[00-分支索引](00-分支索引.md) · [PlanTaskController](../测试计划/PlanTaskController.md) · [ExecuteRecordTaskController](../测试计划/ExecuteRecordTaskController.md)
