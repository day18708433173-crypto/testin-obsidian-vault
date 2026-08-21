---
branch: syy.release.z7.8.1.0
module: real-test
type: 接口文档
entry: WebMvc
---

# TaskController

任务 CRUD 的核心 MVC 控制器，管理测试任务的创建、查询、脚本重置、用户适配、测试计划结果发送等。

类路径：`real-test/src/main/java/cn/testin/mvc/controller/TaskController.java`，基础路径 `/v3/task`。

## 本类接口一览

| 接口 | 方法 | 路径 | 功能 |
| --- | --- | --- | --- |
| taskAdd | POST | /v3/task | 新增测试任务 |
| getTaskInfoByJobId | GET | /v3/task/{job_id} | 根据定时任务 jobId 获取任务详情 |
| getTaskInfoByTaskId | GET | /v3/task/tasks/{task_id} | 根据 taskId 获取任务详情（可选过滤脚本状态） |
| scriptResetTask | POST | /v3/task/tasks/script_reset | 脚本重置任务 |
| updateScriptGroupToScriptNo | POST | /v3/task/tasks_template/script_group_to_script | 模板脚本组更新为脚本编号 |
| getSubSubTaskInfoByCondition | POST | /v3/task/tasks/sub_sub_task_info | 条件查询子子任务信息 |
| getTaskUserAdapt | GET | /v3/task/tasks/user_adapt/{task_id} | 获取任务用户适配信息 |
| sendTestPlanResult | GET | /v3/task/tasks/send_plan/{task_id} | 发送测试计划结果 |

## taskAdd (`POST /v3/task`)

- **实现意图**：接收完整的任务新增请求 `TaskAddRequestDTO`，创建测试任务并返回任务 ID。

- **请求参数**：body `TaskAddRequestDTO`（含设备、脚本、执行标准、通知等完整任务配置）。

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业id |
| projectId | Integer | 否 | 项目id |
| userId | Integer | 否 | 用户id |
| saveTemplate | Integer | 否 | 是否保存为模板 |
| executeRecordTaskId | Long | 否 | 测试计划执行记录任务id |
| executeRecordTaskName | String | 否 | 测试计划执行记录任务名 |
| jobId | Integer | 否 | 模板/定时任务id |
| taskName | String | 否 | 任务名称 |
| taskType | Integer | 否 | 任务类型 |
| bizCode | Integer | 否 | 业务编码 |
| executeType | Integer | 否 | 执行方式 |
| execStandard | JSONObject | 否 | 执行标准 |
| scripts | JSONArray | 否 | 脚本列表 |
| scriptTotal | Integer | 否 | 脚本总数 |
| scriptNos | JSONArray | 否 | 脚本编号列表 |
| checkDevice | Integer | 否 | 是否校验设备 |
| taskDeviceCondition | JSONObject | 否 | 设备全选条件 |
| devices | JSONArray | 否 | 设备列表 |
| deviceIds | JSONArray | 否 | 设备id列表 |
| deviceTotal | Integer | 否 | 设备总数 |
| dataSource | JSONObject | 否 | 数据源配置 |
| networks | Integer | 否 | 网络配置 |
| simulateNetworkName | String | 否 | 模拟网络名称 |
| suiteInfo | JSONObject | 否 | 应用相关数据 |
| quartzInfo | JSONObject | 否 | 定时任务信息 |
| envId | Integer | 否 | 环境id |
| taskNotice | JSONObject | 否 | 通知配置 |
| additionalInfo | String | 否 | 回调附加数据 |
| level | Integer | 否 | 级别 |
| params | String | 否 | 参数 |
| extendedChannel | String | 否 | 扩展渠道 |
| extendedChannelUrl | String | 否 | 扩展渠道地址 |
| callbackUrl | String | 否 | 回调地址 |
| resetInfo | JSONObject | 否 | 重置信息 |
| executeRecordId | Long | 否 | 执行记录id |
| taskReleaseTimePeriodsList | JSONArray | 否 | 任务释放时间段列表 |
| onlyReset | Integer | 否 | 仅重置 |

- **返回参数**：`ResponseResult<BaseResultResponseDTO>`，data 含 `taskId`。

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | String | 创建的任务id（taskId） |

- **处理流程**：TaskController -> TaskService.taskAdd -> 任务入库 + 通知分发。
- **调用链**：[TaskService](TaskService.md) -> `ITaskService` business 层。外部服务：[RealScheduling](../../../任务调度服务（real-scheduling）/07-开放接口文档/00-模块索引.md)（任务调度）、NoticeManager（通知）、[RealControlCenter](../../../设备控制中心（real-controlcenter）/07-开放接口文档/00-模块索引.md)（设备分配）。
- **涉及表与 SQL**：`preal_user_adapt`、`preal_adapt_expand`、`pmreal_adapt_detail`、`pmreal_task_summary` 等。

## getTaskInfoByJobId (`GET /v3/task/{job_id}`)

- **实现意图**：通过定时任务 jobId（整数）获取关联的测试任务详情。

- **请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| job_id | Integer | 是 | 路径变量，模板/定时任务id |

- **返回参数**：`ResponseResult<TaskInfoResponseDTO>`。

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 任务详情（字段同 TaskAddRequestDTO 及关联信息，代码未逐字段确认） |

- **调用链**：[TaskService](TaskService.md) -> `IQuartzJobInfoDAO`。

## getTaskInfoByTaskId (`GET /v3/task/tasks/{task_id}`)

- **实现意图**：通过 taskId 获取任务详情，可选过滤脚本状态（逗号分隔的多状态值）。

- **请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 路径变量，任务id |
| script_status | String | 否 | 脚本状态过滤，逗号分隔（如 "1,2"） |

- **返回参数**：`ResponseResult<TaskInfoResponseDTO>`。

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 任务详情（TaskInfoResponseDTO） |

- **调用链**：[TaskService](TaskService.md).getTaskInfoByTaskId -> DAO 层查询 + 状态过滤。

## scriptResetTask (`POST /v3/task/tasks/script_reset`)

- **实现意图**：对已有任务执行脚本重置操作（如脚本变更后重新下发）。

- **请求参数**：body `ScriptTaskResetRequestDTO`。

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 否 | 项目id |
| userId | Integer | 否 | 用户id |
| userName | String | 否 | 用户名 |
| taskId | String | 否 | 任务id |
| executeRecordTaskId | Long | 否 | 执行记录任务id |
| subSubTaskId | JSONArray | 否 | 子子任务id列表（元素 String） |

- **返回参数**：`ResponseResult<BaseResultResponseDTO>`（含新 taskId）。

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | String | taskId |

- **调用链**：[TaskService](TaskService.md).scriptResetTask -> 重新创建子任务/下发。

## updateScriptGroupToScriptNo (`POST /v3/task/tasks_template/script_group_to_script`)

- **实现意图**：将模板中的脚本组 ID 更新为具体脚本编号。

- **请求参数**：body `TaskTemplateScriptGroupToScriptDTO`（可选）。

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobIds | JSONArray | 否 | 模板/定时任务id列表 |
| repeat | Boolean | 否 | 是否重复 |

- **返回参数**：`ResponseResult<BaseResultResponseDTO>`。

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否成功 |

- **调用链**：[TaskService](TaskService.md).updateScriptGroupToScriptNo。

## getSubSubTaskInfoByCondition (`POST /v3/task/tasks/sub_sub_task_info`)

- **实现意图**：根据条件查询子子任务级别的执行详情列表。

- **请求参数**：body `TaskInfoConditionDTO`。

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| errorCauseMessage | String | 否 | 错误原因信息 |
| taskIds | JSONArray | 否 | 任务id列表 |
| deviceIp | String | 否 | 设备ip |

- **返回参数**：`ResponseResult<ResultListResponseDTO<SubSubTaskInfoResponseDTO>>`。

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 子子任务信息列表，元素为 `SubSubTaskInfoResponseDTO` |
| data.list[].taskId | String | 任务 ID |
| data.list[].subTaskId | String | 子任务 ID |
| data.list[].subSubTaskId | String | 子子任务 ID |

- **调用链**：[TaskService](TaskService.md).getSubSubTaskInfoByCondition。

## getTaskUserAdapt (`GET /v3/task/tasks/user_adapt/{task_id}`)

- **实现意图**：获取指定任务的用户适配信息（提测用户、企业、项目关联）。

- **请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 路径变量，任务id |

- **返回参数**：`ResponseResult<TaskUserAdaptResponseDTO>`。

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.status | Integer | 状态 |
| data.vhost | Integer | vhost |
| data.eid | Integer | 企业id |
| data.userId | Integer | 用户id |
| data.taskId | String | 任务id |
| data.projectId | Integer | 项目id |
| data.appName | String | 应用名 |
| data.packageName | String | 包名 |
| data.packageUrl | String | 包地址 |
| data.appVersion | String | 应用版本 |
| data.testType | Integer | 测试类型 |
| data.execStatus | Integer | 执行状态 |
| data.cancelled | Integer | 是否取消 |
| data.checkApp | Integer | 是否校验应用 |
| data.taskTotal | Integer | 任务总数 |
| data.createTime | Long | 创建时间 |
| data.updateTime | Long | 更新时间 |
| data.sourceTestPlan | Integer | 是否来源测试计划 |

- **调用链**：[TaskService](TaskService.md).getTaskUserAdapt -> `IPrealUserAdaptDAO`。

## sendTestPlanResult (`GET /v3/task/tasks/send_plan/{task_id}`)

- **实现意图**：手动触发发送测试计划执行结果（通常由定时回调触发）。

- **请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 路径变量，任务id |

- **返回参数**：`ResponseResult<BaseResultResponseDTO>`。

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Boolean | 固定返回 true |

- **调用链**：[TaskService](TaskService.md).sendTestPlanResult -> NoticeManager 发送通知。
