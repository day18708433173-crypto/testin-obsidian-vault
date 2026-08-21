---
module: real-test
type: 开放接口文档
source: db_mcfg + 代码
---

# app处理服务 OpenAPI 接口文档

> 代码仓库：`real-test`（分支 `syy.release.z7.8.1.0`）。
> 网关路由：模块 `realtest` → `http://testin-aio-real-test:8080`。
> 双入口：9 个 Spring MVC Controller（V3）+ 20 个 ApiServlet service（V1）。本索引覆盖全部 146 个接口（115 个 V1 op + 31 个 V3 端点），按功能域分组。

## 通用返回结构

- **V1（ApiServlet，action/op）**：`{"code": 0, "msg": "...", "data": {...}}`，`code=0` 表示成功。`data` 内含 `result` / `objInfo` / `list` / `page` / `pageSize` / `totalPage` / `totalRow` 等键，以各接口为准。
- **V3（Spring MVC）**：统一 `ResponseResult<T>` 封装，结构 `{"code": 0, "msg": "...", "data": {...}}`（对应路由表口径中的「BaseResponseDTO」）：

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | T | 业务数据，类型随接口而定（`BaseResponseDTO`=Integer result / `BaseResultResponseDTO`=Object result / `PageResponseDTO` / `ResultListResponseDTO` 等） |

分页类接口 `data` 为 `PageResponseDTO<T>`：

| 字段 | 类型 | 说明 |
|---|---|---|
| page | Integer | 当前页 |
| pageSize | Integer | 每页大小 |
| totalPage | Integer | 总页数 |
| totalRow | Long | 总行数 |
| list | JSONArray | 当前页数据列表 |

## 通用约定

- 转发模式：🟢 = V1 原生（passThroughType=0，`action.op` 短格式）；🔵 = V3 透传（passThroughType=1）；🟡 = V3→V1 转换（passThroughType=0 + special_api_action/op）。
- 鉴权：V1 ApiServlet 接口默认 `needLogin=1`；路由表明确标注 `needLogin=0` 的接口（如 `ParamSource.assign`）已单独标注。V3 接口 `needLogin` 以路由表为准，未收录端点按同模块默认 `needLogin=1` 标注（代码未确认）。
- 「必填」依据：V1 = `reqjson.getXxx/optXxx/isNull` 读取 + `verifyParams`/`CheckArgs`/`Assert` 非空校验；V3 = `@RequestBody` DTO 上的 `@NotNull/@NotBlank/@Valid` 注解、`@RequestParam`（未标 `required=false` 即为必填）、以及 Controller/Service 内显式空值校验。
- V3 路径：网关前缀 `/v3/realtest`（mkey=realtest）+ Controller 的 `@RequestMapping` 相对路径。`ReportController` 类上 `@RequestMapping("report")` 与方法上的 `@GetMapping("report/...")` 叠加，代码实际路径为 `report/report/...`。
- 路由表 V3 端点中，`GET /v3/realtest/report/steps`（标注指向 ReportController）在代码中未找到对应 `/steps` 端点，标记「（代码未确认）」。

---

## 一、测试计划与任务（64）

### 测试计划与任务 · TaskController（WebMvc `/task`，8 接口）

#### 创建任务 `POST /v3/realtest/task`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realtest/task` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 新增测试任务（含模板/执行记录/设备/脚本/数据源等完整配置），返回 taskId |

**请求参数**（`TaskAddRequestDTO`，均无校验注解）

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

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | String | 创建的任务id（taskId） |

> 代码出处：`TaskController.taskAdd`

#### 按模板id获取任务 `GET /v3/realtest/task/{job_id}`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realtest/task/{job_id}` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 按 jobId（模板/定时任务id）获取任务详情 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| job_id | Integer | 是 | 路径变量，模板/定时任务id |

**返回参数**（`data` = `TaskInfoResponseDTO`，镜像 `TaskAddRequestDTO` 各字段 + 关联信息）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 任务详情（字段同 TaskAddRequestDTO 及关联信息，代码未逐字段确认） |

> 代码出处：`TaskController.getTaskInfoByJobId`

#### 按 taskId 获取任务 `GET /v3/realtest/task/tasks/{task_id}`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realtest/task/tasks/{task_id}` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 按 taskId 获取任务详情，支持 script_status 过滤脚本状态 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 路径变量，任务id |
| script_status | String | 否 | 脚本状态过滤，逗号分隔（如 "1,2"） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 任务详情（TaskInfoResponseDTO） |

> 代码出处：`TaskController.getTaskInfoByTaskId`

#### 脚本重置 `POST /v3/realtest/task/tasks/script_reset`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realtest/task/tasks/script_reset` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 脚本重置（补测脚本），返回 taskId |

**请求参数**（`ScriptTaskResetRequestDTO`）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 否 | 项目id |
| userId | Integer | 否 | 用户id |
| userName | String | 否 | 用户名 |
| taskId | String | 否 | 任务id |
| executeRecordTaskId | Long | 否 | 执行记录任务id |
| subSubTaskId | JSONArray | 否 | 子子任务id列表（元素 String） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | String | taskId |

> 代码出处：`TaskController.scriptResetTask`

#### 脚本组转脚本编号 `POST /v3/realtest/task/tasks_template/script_group_to_script`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realtest/task/tasks_template/script_group_to_script` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 脚本组转脚本编号（模板场景） |

**请求参数**（`TaskTemplateScriptGroupToScriptDTO`，请求体可空）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobIds | JSONArray | 否 | 模板/定时任务id列表 |
| repeat | Boolean | 否 | 是否重复 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Boolean | 是否成功 |

> 代码出处：`TaskController.updateScriptGroupToScriptNo`

#### 子子任务信息 `POST /v3/realtest/task/tasks/sub_sub_task_info`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realtest/task/tasks/sub_sub_task_info` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 按条件查询子子任务信息 |

**请求参数**（`TaskInfoConditionDTO`）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| errorCauseMessage | String | 否 | 错误原因信息 |
| taskIds | JSONArray | 否 | 任务id列表 |
| deviceIp | String | 否 | 设备ip |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 子子任务信息列表，元素为 `SubSubTaskInfoResponseDTO` |
| data.list[].taskId | String | 任务 ID |
| data.list[].subTaskId | String | 子任务 ID |
| data.list[].subSubTaskId | String | 子子任务 ID |

> 代码出处：`TaskController.getSubSubTaskInfoByCondition`

#### 任务用户适配 `GET /v3/realtest/task/tasks/user_adapt/{task_id}`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realtest/task/tasks/user_adapt/{task_id}` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 获取任务用户适配信息 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 路径变量，任务id |

**返回参数**（`data` = `TaskUserAdaptResponseDTO`）

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

> 代码出处：`TaskController.getTaskUserAdapt`

#### 测试计划结果发送 `GET /v3/realtest/task/tasks/send_plan/{task_id}`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realtest/task/tasks/send_plan/{task_id}` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 发送测试计划结果 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 路径变量，任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Boolean | 固定返回 true |

> 代码出处：`TaskController.sendTestPlanResult`

### 测试计划与任务 · TestPlanController（WebMvc `/realtest`，2 接口）

#### 测试计划 Excel 导出 `GET /v3/realtest/plan/excel`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realtest/plan/excel` |
| 鉴权 | needLogin=1 |
| 说明 | 按执行记录导出测试计划 Excel 报告 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| record_id | Long | 否 | 执行记录id |
| user_id | Integer | 否 | 用户id |

**返回参数**（`ResponseResult`，`data` 为导出结果，结构代码未确认）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | Object | Excel 导出结果（代码未确认） |

> 代码出处：`TestPlanController.excel`

#### 测试计划 Excel 导出（分享） `GET /v3/realtest/plan/excel/share`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realtest/plan/excel/share` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 通过分享密钥导出测试计划 Excel（校验 skey 与 execTaskId 匹配） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| exec_task_id | String | 是 | 执行任务id（空则返回参数错误） |
| share_id | String | 是 | 分享密钥 skey（空则返回参数错误） |
| record_id | Long | 是 | 执行记录id |
| user_id | Integer | 否 | 用户id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 / 500 参数无效 |
| msg | String | 提示信息 |
| data | Object | Excel 导出结果（代码未确认） |

> 代码出处：`TestPlanController.shareExcel`

### 测试计划与任务 · TaskReportController（WebMvc `/realtest`，3 接口）

#### 任务报告详情 `POST /v3/realtest/report/detail`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realtest/report/detail` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 获取任务报告详情（脚本信息含行号） |

**请求参数**（`TaskReportRequestDTO`）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskId | String | 否 | 任务id |
| scriptStatus | Integer | 否 | 脚本状态 |
| paramSource | String | 否 | 参数来源 |
| newTagList | JSONArray | 否 | 新标签列表 |
| keyWord | String | 否 | 关键字 |

**返回参数**（`data` = `ScriptInfoWithRowIds`）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.scriptDataDetailUuidWithRowIds | JSONArray | 脚本数据详情（含行id） |
| data.scriptNoWithRowIds | JSONArray | 脚本编号（含行id） |
| data.normalScriptNoDeviceIdRowId | JSONArray | 普通脚本编号-设备id-行id |
| data.deviceIds | JSONArray | 设备id列表 |
| data.reportDetailMap | JSONObject | 报告详情映射 |

> 代码出处：`TaskReportController.getTaskReportDetailInfo`

#### 任务报告汇总（东北证券定制） `POST /v3/realtest/report/summarylist`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realtest/report/summarylist` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 任务报告汇总列表（仅东北证券调用） |

**请求参数**（`TaskReportRequestDTO`，同 report/detail）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskId | String | 否 | 任务id |
| scriptStatus | Integer | 否 | 脚本状态 |
| paramSource | String | 否 | 参数来源 |
| newTagList | JSONArray | 否 | 新标签列表 |
| keyWord | String | 否 | 关键字 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | JSONArray | 汇总数据（List<Map<String,Object>>） |

> 代码出处：`TaskReportController.summarylist`

#### 更新适配用户 `POST /v3/realtest/update`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realtest/update` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 更新适配用户（当前实现为固定返回 1） |

**请求参数**：无。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | Integer | 固定返回 1 |

> 代码出处：`TaskReportController.updateAdaptUser`

### 测试计划与任务 · QuartzJobStatementController（WebMvc `/quartz_job_statement`，1 接口）

#### 按 taskId 删除定时任务记录 `DELETE /v3/realtest/quartz_job_statement/remove_by_task_id`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | DELETE |
| 路径 | `/v3/realtest/quartz_job_statement/remove_by_task_id` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 按 taskId 删除定时任务执行记录 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | Integer | 删除结果 |

> 代码出处：`QuartzJobStatementController.removeByTaskId`

### 测试计划与任务 · Task（ApiServlet action=app，28 op）

#### add — 新增测试任务

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.add` |
| 鉴权 | needLogin=1 |
| 说明 | 新增测试任务（含参数校验、用户适配、设备适配），支持 taskTemplateId 模板创建 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目组id |
| userid | Integer | 是 | 用户id |
| bizCode | Integer | 是 | 业务编码 |
| taskDescr | String | 是 | 任务描述 |
| serialRun | Integer | 否 | 串并行标志 |
| appinfo | JSONObject | 否 | 应用信息 |
| devices | JSONArray | 是 | 设备列表 |
| execStandard | JSONObject | 是 | 执行标准 |
| taskTemplateId | Integer | 否 | 任务模板id（>0 时走模板创建） |
| initPwd | String | 否 | 快速提测用户密码 |
| quotaCode | Integer | 否 | 第三方接入提测业务编码 |
| showCompatibleReportUrl | String | 否 | 是否显示报告邮件中的在线报告 URL |
| testChannel | String | 否 | 提测渠道 |
| crossTaskId | String | 否 | 多端任务 ID |
| userName | String | 否 | 用户名（未传时按 userid 回查填充） |
| paramSourceName | String | 否 | 参数来源名称（未传时按 paramSource 回查填充） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | String | 创建的任务 ID（taskId） |

> 代码出处：`Task.add`

#### cancel — 取消测试任务

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.cancel` |
| 鉴权 | needLogin=1 |
| 说明 | 取消指定任务（支持子任务级别取消、欠费取消码区分） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目组id |
| userid | Integer | 是 | 用户id |
| taskid | String | 是 | 任务id |
| subtaskid | String | 否 | 子任务id |
| cancelCode | Integer | 否 | 取消码 |
| taskGroup | JSONObject | 否 | 恒生任务组 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Task.cancel`

#### batchCancel — 批量取消测试任务

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.batchCancel` |
| 鉴权 | needLogin=1 |
| 说明 | 批量取消测试任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目组id |
| userid | Integer | 是 | 用户id |
| taskids | JSONArray | 是 | 任务id列表 |
| cancelCode | Integer | 否 | 取消码 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 成功取消的任务数量 |

> 代码出处：`Task.batchCancel`

#### cancelWithNoUserId — 无 userId 取消（恒生专用）

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.cancelWithNoUserId` |
| 鉴权 | needLogin=1 |
| 说明 | 无 userId 取消任务（恒生专用） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目组id |
| taskid | String | 是 | 任务id |
| subtaskid | String | 否 | 子任务id |
| cancelCode | Integer | 否 | 取消码 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Task.cancelWithNoUserId`

#### details — 获取任务详情

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.details` |
| 鉴权 | needLogin=1 |
| 说明 | 获取任务完整详情（含 eid/projectid/userid 校验 + skey 分享支持） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥（与 taskid 二选一） |
| eid | Integer | 是 | 企业id（补充后仍 null/<=0 返回 paraInvalid） |
| projectid | Integer | 是 | 项目id |
| userid | Integer | 是 | 用户id |
| userprojectids | JSONArray | 否 | 用户项目 ID 列表（分享场景权限补充） |
| subtaskid | String | 否 | 子任务 ID |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | 任务详情（PmrealAdaptDetail 等扩展信息，代码未逐字段确认） |

> 代码出处：`Task.details`

#### overview — 获取任务概况

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.overview` |
| 鉴权 | needLogin=1 |
| 说明 | 获取任务概况信息 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目组id |
| userid | Integer | 是 | 用户id |
| taskid | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | 任务概况对象 |

> 代码出处：`Task.overview`

#### complete — 完善任务应用信息（应用检测回调）

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.complete` |
| 鉴权 | needLogin=1 |
| 说明 | 完善任务应用信息（应用检测回调） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目组id |
| taskid | String | 是 | 任务id |
| errorCode | Integer | 是 | 错误码 |
| suiteId | Integer | 否 | 应用 ID（跨平台） |
| msg | String | 否 | 错误信息（检测失败原因） |
| appinfos | JSONArray | 否 | 应用信息列表（msg 为空时必填，null 或空返回 paraInvalid） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Task.complete`

#### getUserAdapt / userAdapt — 获取用户适配信息

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.getUserAdapt`（别名 `userAdapt`） |
| 鉴权 | needLogin=1 |
| 说明 | 获取任务用户适配信息 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| projectid | Integer | 否 | 项目id |
| userid | Integer | 否 | 用户id |
| eid | Integer | 否 | 企业id |
| userprojectids | JSONArray | 否 | 用户项目 ID 列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | 用户适配信息（PrealUserAdapt） |

> 代码出处：`Task.getUserAdapt`

#### reportVideo — 同步视频信息到报告

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.reportVideo` |
| 鉴权 | needLogin=1 |
| 说明 | 同步视频信息到报告 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（代码未校验） |
| subtaskid | String | 否 | 子任务id |
| subsubtaskid | String | 否 | 子子任务id |
| videoUrl | String | 否 | 视频地址 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Task.reportVideo`

#### retest — 设备级任务补测

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.retest` |
| 鉴权 | needLogin=1 |
| 说明 | 设备级补测，需提供新设备列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目组id |
| taskid | String | 是 | 任务id |
| subtaskid | String | 是 | 子任务id |
| devices | JSONArray | 是 | 新设备列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Task.retest`

#### monitorTaskMaintain — 监控任务变更上报

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.monitorTaskMaintain` |
| 鉴权 | needLogin=1 |
| 说明 | 监控任务变更上报 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| monitorId | Long | 是 | 监控id（null 返回 paraInvalid） |
| tasks | JSONArray | 否 | 任务列表 |
| jobNames | JSONArray | 否 | 定时任务名列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Task.monitorTaskMaintain`

#### detail — 获取任务详情（支持 keywords 过滤）

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.detail` |
| 鉴权 | needLogin=1 |
| 说明 | 获取任务详情（AppPrx.get 替代，支持 keywords 过滤） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| projectid | Integer | 否 | 项目id |
| userid | Integer | 否 | 用户id |
| eid | Integer | 否 | 企业id |
| userprojectids | JSONArray | 否 | 用户项目 ID 列表 |
| keywords | JSONArray | 否 | 过滤关键字（元素 String，自动追加 scriptTotal/scriptNewTotal） |
| scriptStatuses | JSONArray | 否 | 脚本状态过滤（元素 Integer） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | PmrealAdaptDetail 详情对象 |

> 代码出处：`Task.detail`

#### scriptRetest — 脚本级补测

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.scriptRetest` |
| 鉴权 | needLogin=1 |
| 说明 | 脚本级补测 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目组id |
| taskid | String | 是 | 任务id |
| subtaskid | String | 是 | 子任务id |
| subsubtasks | JSONArray | 是 | 子子任务id列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 补测结果（iRetestService.scriptRetest 返回值） |

> 代码出处：`Task.scriptRetest`

#### getRealStatSummaryDetail — 获取任务统计摘要详情

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.getRealStatSummaryDetail` |
| 鉴权 | needLogin=1 |
| 说明 | 获取任务统计摘要（设备分布/性能统计/抽查摘要/执行详情，含分页） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| conditions | String | 否 | 查询条件（JSON 字符串，round/item/releaseVer/dpiWidth/dpiHeight/aliasName 等） |
| keywords | JSONArray | 否 | 关键字（设备分布/性能统计/抽查摘要等） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | 任务统计摘要（PmrealStatSummary） |
| data.objInfo.spotTestSummarys | JSONArray | 抽查摘要列表（元素 PmrealSpotTestSummary，代码未确认字段） |
| data.objInfo.deviceExecInfos | JSONArray | 设备执行详情（元素 DeviceExecInfo，分页后，代码未确认字段） |

> 代码出处：`Task.getRealStatSummaryDetail`

#### share — 报告分享（生成/更新 skey）

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢（另有 V3→V1 转换 `GET /v3/realtest/task/share`） |
| HTTP 方法 | POST |
| 路径 | `app.Task.share` |
| 鉴权 | needLogin=1 |
| 说明 | 生成或获取任务分享密钥 skey（MD5 生成，重试防冲突） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| userid | Integer | 是 | 用户id |
| eid | Integer | 是 | 企业id |
| type | Integer | 否 | 分享类型（ShareConfig 枚举） |
| projectid | Integer | 否 | 项目组id |
| userprojectids | JSONArray | 否 | 用户项目 ID 列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | String | skey 分享密钥 |

> 代码出处：`Task.share`

#### shareInfo — 获取任务分享信息

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.shareInfo` |
| 鉴权 | needLogin=1 |
| 说明 | 获取任务分享信息 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | 分享信息（RealReportShare） |
| data.objInfo.skey | String | 短地址密钥 |
| data.objInfo.taskid | String | 任务 ID |
| data.objInfo.reportKey | String | 查看报告 key |
| data.objInfo.status | Integer | 状态 |
| data.objInfo.createtime | Long | 创建时间 |
| data.objInfo.updatetime | Long | 更新时间 |
| data.objInfo.content | JSONObject | 分享内容 |

> 代码出处：`Task.shareInfo`

#### maintainReportSummary — 维护报告总结

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.maintainReportSummary` |
| 鉴权 | needLogin=1 |
| 说明 | 维护报告总结（清除旧 Excel 连接后更新报告摘要） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| userid | Integer | 是 | 用户id |
| context | String | 否 | 报告总结内容（默认空串） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 更新影响行数（0 或 1） |
| data.userName | String | 用户名（更新成功时返回） |

> 代码出处：`Task.maintainReportSummary`

#### verification — 验证 taskid/skey 有效性

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.verification` |
| 鉴权 | needLogin=1 |
| 说明 | 验证 taskid/skey 有效性 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| userid | Integer | 否 | 用户id |
| eid | Integer | 否 | 企业id |
| userprojectids | JSONArray | 否 | 用户项目 ID 列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 固定返回 1（验证通过） |

> 代码出处：`Task.verification`

#### getScriptParams — 获取脚本参数

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.getScriptParams` |
| 鉴权 | needLogin=1 |
| 说明 | 获取脚本参数（AppPrx.getScriptParams 替代） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| reqid | String | 是 | 请求id |

**返回参数**（原始 JSONObject）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 |
| status | Integer | 状态 |
| msg | String | 提示信息 |
| createtime | Long | 创建时间 |
| updatetime | Long | 更新时间 |
| resContent | JSONObject | 结果内容（已移除 scriptsummary 节点） |

> 代码出处：`Task.getScriptParams`

#### repeatTest — 批量补测

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.repeatTest` |
| 鉴权 | needLogin=1 |
| 说明 | 批量补测（支持设备补测 + 脚本补测） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| subtasks | JSONArray | 是 | 子任务列表 |
| devices | JSONArray | 否 | 设备补测列表 |
| subsubtaskinfo | JSONObject | 否 | 脚本补测信息 |
| appInfo | JSONObject | 否 | 应用信息 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Task.repeatTest`

#### initData — 初始化提测数据

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.initData` |
| 鉴权 | needLogin=1 |
| 说明 | 无 App 快速提测（初始化提测数据） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| bizCode | Integer | 是 | 业务编码 |
| scripts | JSONArray | 是 | 脚本列表 |
| taskDescr | String | 否 | 任务描述 |
| additionalInfo | String | 否 | 附加信息 |
| callbackUrl | String | 否 | 回调地址 |
| extendedChannel | String | 否 | 扩展渠道 |
| noApp | Integer | 否 | 无 App 标志 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | String | reqid 请求id |

> 代码出处：`Task.initData`

#### getInitData — 获取初始化提测结果

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.getInitData` |
| 鉴权 | needLogin=1 |
| 说明 | 从 Redis 查询无 App 快速提测结果 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| reqId | String | 是 | 请求id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | JSONObject | 初始化结果（InitTask） |
| data.result.taskDescr | String | 任务描述 |
| data.result.additionalInfo | String | 附加信息 |
| data.result.callbackUrl | String | 回调地址 |
| data.result.extendedChannel | String | 扩展渠道 |
| data.result.noApp | Integer | 无 App 标志 |
| data.result.bizCode | Integer | 业务编码 |
| data.result.scripts | JSONArray | 脚本列表（元素 PmrealAdaptScript） |

> 代码出处：`Task.getInitData`

#### getSubTestMenu — 获取子测试菜单配置

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.getSubTestMenu` |
| 鉴权 | needLogin=1 |
| 说明 | 获取子测试菜单配置 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| extendedChannel | String | 否 | 扩展渠道 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | JSONObject | 菜单配置 |
| data.result.eid | Integer | 企业 ID |
| data.result.config | JSONArray | 步骤配置列表（元素 key/name/status） |

> 代码出处：`Task.getSubTestMenu`

#### sendEmail — 重新发送结果邮件

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.sendEmail` |
| 鉴权 | needLogin=1 |
| 说明 | 手动重新发送任务结果邮件通知 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 结果 |

> 代码出处：`Task.sendEmail`

#### execute — 触发任务执行

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.execute` |
| 鉴权 | needLogin=1 |
| 说明 | 触发任务执行 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 结果 |

> 代码出处：`Task.execute`

#### pause — 暂停任务下发

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.pause` |
| 鉴权 | needLogin=1 |
| 说明 | 暂停任务下发 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 结果 |

> 代码出处：`Task.pause`

#### resume — 恢复任务下发

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Task.resume` |
| 鉴权 | needLogin=1 |
| 说明 | 恢复任务下发 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 结果 |

> 代码出处：`Task.resume`

### 测试计划与任务 · ScheduledJob（ApiServlet action=app，15 op）

#### get — 获取单个定时任务详情

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ScheduledJob.get` |
| 鉴权 | needLogin=1 |
| 说明 | 获取定时任务详情（jobInfo + jobStatement + 关联 taskIds） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| jobId | Integer | 是 | 定时任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | 定时任务详情 |

> 代码出处：`ScheduledJob.get`

#### list — 分页查询定时任务列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ScheduledJob.list` |
| 鉴权 | needLogin=1 |
| 说明 | 分页条件查询定时任务列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| page | Integer | 是 | 当前页 |
| pageSize | Integer | 是 | 每页大小 |
| bizCode | Integer | 否 | 业务编码 |
| appName | String | 否 | 应用名 |
| appVersion | String | 否 | 应用版本 |
| taskDesc | String | 否 | 任务描述 |
| userName | String | 否 | 用户名 |
| syspfId | Integer | 否 | 系统平台id |
| appId | Integer | 否 | 应用id |
| channelId | String | 否 | 渠道id |
| suiteId | Integer | 否 | Suite id |
| jobStatus | String | 否 | 任务状态 |
| userId | Integer | 否 | 用户id |
| pkgId | Integer | 否 | 包id |
| packageName | String | 否 | 包名 |
| jobType | Integer | 否 | 任务类型 |
| orderByCol | String | 否 | 排序字段 |
| orderByType | String | 否 | 排序方式 |
| startTime | Long | 否 | 开始时间 |
| endTime | Long | 否 | 结束时间 |
| dirId | Integer | 否 | 目录id |
| jobIds | JSONArray | 否 | 任务id列表 |
| jobStatuses | JSONArray | 否 | 任务状态列表 |
| noJobIds | JSONArray | 否 | 排除任务id列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 定时任务列表，元素为 DbQuartzJobInfo |
| data.list[].relations | JSONArray | 关联测试计划信息（元素 TaskRecordsDTO） |
| data.list[].effectiveDeviceTotal | Integer | 有效设备数量 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |

> 代码出处：`ScheduledJob.list`

#### execute — 立即触发定时任务执行

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ScheduledJob.execute` |
| 鉴权 | needLogin=1 |
| 说明 | 立即触发定时任务执行 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 否 | 项目id（代码未显式校验，实际执行需传入） |
| jobId | Integer | 否 | 定时任务id（代码未显式校验，实际执行需传入） |
| share | Integer | 否 | 是否分享报告（默认 0） |
| extendedChannel | String | 否 | 扩展渠道信息 |
| extendedChannelUrl | String | 否 | 扩展渠道 URL |
| callbackUrl | String | 否 | 回调 URL |
| additionalInfo | String | 否 | 附加信息 |
| userOnline | JSONObject | 否 | 在线执行用户信息（exeUser） |
| isManualExecution | Integer | 否 | 是否手动执行 |
| taskDescr | String | 否 | 任务描述（替换） |
| userid | Integer | 否 | 用户 ID |
| appinfo | JSONObject | 否 | 应用信息 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | String | 生成的 taskId（执行结果标识） |

> 代码出处：`ScheduledJob.execute`

#### batchExecute — 批量触发定时任务执行

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ScheduledJob.batchExecute` |
| 鉴权 | needLogin=1 |
| 说明 | 批量触发定时任务执行 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobIds | JSONArray | 是 | 定时任务id列表（元素 Integer，null 或空返回 GeneralException） |
| projectid | Integer | 否 | 项目id |
| share | Integer | 否 | 是否分享报告（默认 0） |
| extendedChannel | String | 否 | 扩展渠道信息 |
| extendedChannelUrl | String | 否 | 扩展渠道 URL |
| callbackUrl | String | 否 | 回调 URL |
| additionalInfo | String | 否 | 附加信息 |
| userOnline | JSONObject | 否 | 在线执行用户信息 |
| isManualExecution | Integer | 否 | 是否手动执行 |
| taskDescr | String | 否 | 任务描述（替换） |
| userid | Integer | 否 | 用户 ID |
| appinfo | JSONObject | 否 | 应用信息 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | JSONArray | 各任务执行结果 taskId 列表，元素 String |

> 代码出处：`ScheduledJob.batchExecute`

#### pause — 暂停定时任务调度

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ScheduledJob.pause` |
| 鉴权 | needLogin=1 |
| 说明 | 暂停定时任务调度 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| jobId | Integer | 是 | 定时任务id |
| userid | Integer | 否 | 用户id（记录修改人） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`ScheduledJob.pause`

#### batchPause — 批量暂停定时任务

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ScheduledJob.batchPause` |
| 鉴权 | needLogin=1 |
| 说明 | 批量暂停定时任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| jobIds | JSONArray | 是 | 定时任务id列表（元素 Integer，null 或空返回 GeneralException） |
| userid | Integer | 否 | 用户id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 成功暂停数量 |

> 代码出处：`ScheduledJob.batchPause`

#### stop — 停止定时任务

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ScheduledJob.stop` |
| 鉴权 | needLogin=1 |
| 说明 | 停止定时任务（删除 Quartz Trigger，任务配置保留） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| jobId | Integer | 是 | 定时任务id |
| eid | Integer | 否 | 企业id |
| userid | Integer | 否 | 用户id |
| username | String | 否 | 用户名 |
| deleteRecords | Integer | 否 | 是否同时删除所有执行记录（默认 0） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`ScheduledJob.stop`

#### batchStop — 批量停止定时任务

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ScheduledJob.batchStop` |
| 鉴权 | needLogin=1 |
| 说明 | 批量停止定时任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| jobIds | JSONArray | 是 | 定时任务id列表（元素 Integer，null 或空返回 GeneralException） |
| eid | Integer | 否 | 企业id |
| userid | Integer | 否 | 用户id |
| username | String | 否 | 用户名 |
| deleteRecords | Integer | 否 | 是否同时删除所有执行记录（默认 0） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 成功停止数量 |

> 代码出处：`ScheduledJob.batchStop`

#### resume — 恢复定时任务调度

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ScheduledJob.resume` |
| 鉴权 | needLogin=1 |
| 说明 | 恢复定时任务调度 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| jobId | Integer | 是 | 定时任务id |
| userid | Integer | 否 | 用户id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`ScheduledJob.resume`

#### batchResume — 批量恢复定时任务

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ScheduledJob.batchResume` |
| 鉴权 | needLogin=1 |
| 说明 | 批量恢复定时任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| jobIds | JSONArray | 是 | 定时任务id列表（元素 Integer，null 或空返回 GeneralException） |
| userid | Integer | 否 | 用户id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 成功恢复数量 |

> 代码出处：`ScheduledJob.batchResume`

#### maintain — 新增或更新定时任务

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ScheduledJob.maintain` |
| 鉴权 | needLogin=1 |
| 说明 | 新增或更新定时任务（cron、设备筛选、脚本、通知），支持模板创建 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| jobId | Integer | 是 | 定时任务id（更新时传入） |
| taskDesc | String | 否 | 任务描述 |
| jobRule | String | 否 | 定时任务规则（JSON 字符串） |
| taskContent | String | 否 | 提测内容 |
| isBand | Integer | 否 | 是否绑定任务操作（默认 0） |
| userId | Integer | 否 | 用户id |
| bizCode | Integer | 否 | 业务编码 |
| cronExpression | String | 否 | cron 表达式 |
| devices | JSONArray | 否 | 设备筛选条件 |
| scripts | JSONArray | 否 | 脚本配置 |
| noticeConfig | JSONObject | 否 | 通知设置 |
| jobName | String | 否 | 任务名称 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`ScheduledJob.maintain`

#### getScheduleTaskScriptIds — 获取定时任务关联脚本 ID

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ScheduledJob.getScheduleTaskScriptIds` |
| 鉴权 | needLogin=1 |
| 说明 | 获取定时任务关联的脚本 ID |

**请求参数**：无。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 脚本 ID 列表，元素 Integer |

> 代码出处：`ScheduledJob.getScheduleTaskScriptIds`

#### listTaskIdByJobId — 根据 jobId 获取最近任务 ID 列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ScheduledJob.listTaskIdByJobId` |
| 鉴权 | needLogin=1 |
| 说明 | 根据 jobId 获取最近任务 ID 列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobId | Long | 否 | 定时任务id（代码未校验 null） |
| pageNo | Integer | 否 | 页码（默认 1） |
| pageSize | Integer | 否 | 每页大小（默认 15） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 任务 ID 列表 |
| data.totalRow | Long | 总记录数 |

> 代码出处：`ScheduledJob.listTaskIdByJobId`

#### listAllTaskIdByJobId — 获取 jobId 所有关联 taskId

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ScheduledJob.listAllTaskIdByJobId` |
| 鉴权 | needLogin=1 |
| 说明 | 获取 jobId 所有关联 taskId |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobId | Long | 否 | 定时任务id（代码未校验 null） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 任务 ID 列表 |
| data.totalRow | Long | 总记录数 |

> 代码出处：`ScheduledJob.listAllTaskIdByJobId`

#### delTaskScheduledJob — 删除定时任务关联

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ScheduledJob.delTaskScheduledJob` |
| 鉴权 | needLogin=1 |
| 说明 | 删除定时任务关联 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（代码未校验 null） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 删除影响行数 |

> 代码出处：`ScheduledJob.delTaskScheduledJob`

### 测试计划与任务 · Quartz（ApiServlet action=app，5 op）

#### appNameList — 获取项目下应用名列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Quartz.appNameList` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 获取项目下已测试过的应用名列表（下拉框） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| bizCode | Integer | 否 | 业务编码 |
| suiteId | Integer | 否 | Suite id |
| syspfId | Integer | 否 | 系统平台id |
| jobIds | JSONArray | 否 | 任务id列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 应用名列表 |

> 代码出处：`Quartz.appNameList`

#### appChannelList — 获取应用渠道列表（@Deprecated）

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Quartz.appChannelList` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 获取应用渠道列表（已废弃） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| bizCode | Integer | 是 | 业务编码 |
| suiteId | Integer | 否 | Suite id |
| appid | Integer | 否 | 应用id |
| appVersion | String | 否 | 应用版本 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 渠道列表 |

> 代码出处：`Quartz.appChannelList`

#### listQuartzJobByNames — 根据名称列表查询定时任务

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Quartz.listQuartzJobByNames` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 根据定时任务名称列表批量查询任务详情 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobNames | JSONArray | 是 | 定时任务名称列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 定时任务列表 |

> 代码出处：`Quartz.listQuartzJobByNames`

#### suiteNameList — 获取项目下 Suite 名列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Quartz.suiteNameList` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 获取项目下 Suite（应用集）名列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| bizCode | Integer | 否 | 业务编码 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | Suite 名列表 |

> 代码出处：`Quartz.suiteNameList`

#### appVersionList — 获取应用版本列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Quartz.appVersionList` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 获取指定应用的所有版本号列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| bizCode | Integer | 是 | 业务编码 |
| suiteId | Integer | 否 | Suite id |
| syspfId | Integer | 否 | 系统平台id |
| appid | Integer | 否 | 应用id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 版本号列表 |

> 代码出处：`Quartz.appVersionList`

### 测试计划与任务 · TestResult（ApiServlet action=app，1 op）

#### report — 测试结果上报

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.TestResult.report` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 测试结果上报 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskResult | JSONObject | 是 | 任务结果 |
| resultFiles | JSONObject | 否 | 结果文件 |
| appMd5 | String | 否 | 应用 md5 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`TestResult.report`

### 测试计划与任务 · TestProcess（ApiServlet action=app，1 op）

#### report — 测试过程数据上报

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.TestProcess.report` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 测试过程数据上报 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskAction | String | 是 | 任务动作 |
| content | JSONObject | 否 | 内容（taskAction=report 时必填） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`TestProcess.report`

---

## 二、报告与导出（45）

### 报告与导出 · ReportController（WebMvc `/report`，4 接口）

> 注：类上 `@RequestMapping("report")` 与方法 `@GetMapping("report/...")` 叠加，代码实际路径为 `report/report/...`，以下按代码实际路径标注。

#### 查询补测历史记录 `GET /v3/realtest/report/report/{task_id}`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realtest/report/report/{task_id}` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 查询补测历史记录（分页） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 路径变量，任务id |
| page | Integer | 是 | 当前页 |
| page_size | Integer | 是 | 每页大小 |
| sub_sub_task_id | String | 是 | 子子任务id |
| start_time | Long | 否 | 开始时间 |
| end_time | Long | 否 | 结束时间 |

**返回参数**（`data` = `PageResponseDTO<ReTestInfoDTO>`）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |
| data.list | JSONArray | 补测历史列表（ReTestInfoDTO，字段代码未确认） |

> 代码出处：`ReportController.list`

#### 根据 taskIds 查询报告 `POST /v3/realtest/report/report/list`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realtest/report/report/list` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 根据 taskIds 查询报告列表 |

**请求参数**（`ReportListRequestDTO`）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 否 | 项目id |
| taskIds | JSONArray | 否 | 任务id列表 |
| subSubTaskIds | JSONArray | 否 | 子子任务id列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 报告列表（EsReportSummary） |

> 代码出处：`ReportController.list`

#### 报告汇总 `POST /v3/realtest/report/report/summary`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realtest/report/report/summary` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 报告汇总 |

**请求参数**（`ReportListRequestDTO`，同 report/list）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 否 | 项目id |
| taskIds | JSONArray | 否 | 任务id列表 |
| subSubTaskIds | JSONArray | 否 | 子子任务id列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 汇总列表（EsReportSummary） |

> 代码出处：`ReportController.summary`

#### 修改错误类型 `POST /v3/realtest/report/report/modify_error_cause_type`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realtest/report/report/modify_error_cause_type` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 更新错误类型 |

**请求参数**（`MaintainErrorCauseTypeDTO`，继承 `BaseQueryRequestDTO`）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业id |
| projectId | Integer | 否 | 项目id |
| userId | Integer | 否 | 用户id |
| userName | String | 否 | 用户名 |
| errorCauseTypeId | Integer | 否 | 错误类型id |
| taskId | String | 否 | 任务id |
| subTaskId | String | 否 | 子任务id |
| subSubTaskId | String | 否 | 子子任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 更新结果 |

> 代码出处：`ReportController.modifyErrorCauseType`

### 报告与导出 · TemplateController（WebMvc `/realtest/template`，6 接口）

#### 模板分页查询 `POST /v3/realtest/template/templates`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realtest/template/templates` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 分页条件查询任务模板列表 |

**请求参数**（`TemplateRequestDTO`，继承 `PageRequestDTO`）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |
| projectId | Integer | 否 | 项目id |
| suiteId | Integer | 否 | Suite id |
| taskName | String | 否 | 任务名称 |
| userIds | JSONArray | 否 | 用户id列表 |
| createStartTime | Long | 否 | 创建开始时间 |
| createEndTime | Long | 否 | 创建结束时间 |
| filterIds | JSONArray | 否 | 过滤id列表 |
| ids | JSONArray | 否 | id列表 |
| plan | Boolean | 否 | 是否测试计划 |
| needContent | Boolean | 否 | 是否需要内容 |
| needScriptDetail | Boolean | 否 | 是否需要脚本详情 |
| needBaseInfo | Boolean | 否 | 是否需要基础信息 |
| needScriptAndDeviceBashInfo | Boolean | 否 | 是否需要脚本设备基础信息 |
| needDataSourceDetail | Boolean | 否 | 是否需要数据源详情 |
| checkStatus | Boolean | 否 | 校验状态 |

**返回参数**（`data` = `PageResponseDTO<TemplateResponseDTO>`）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |
| data.list | JSONArray | 模板列表（TemplateResponseDTO） |

> 代码出处：`TemplateController.listByCondition`

#### 批量删除模板 `DELETE /v3/realtest/template/batch_delete`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | DELETE |
| 路径 | `/v3/realtest/template/batch_delete` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 批量删除任务模板 |

**请求参数**（`TemplateRemoveRequestDTO`）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| templateIds | JSONArray | 否 | 模板id列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | Integer | 删除结果 |

> 代码出处：`TemplateController.batchRemove`

#### 复制 app 任务 `GET /v3/realtest/template/copy`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realtest/template/copy` |
| 鉴权 | needLogin=1 |
| 说明 | 复制 app 任务（按模板/任务复制） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| job_id | Integer | 是 | 模板/任务id（@Validated） |
| user_id | Integer | 是 | 用户id |
| user_name | String | 是 | 用户名 |
| dir_id | Integer | 是 | 目录id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 复制结果 |

> 代码出处：`TemplateController.copy`

#### 转模板 `GET /v3/realtest/template/trans_template`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realtest/template/trans_template` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 将任务转为模板 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| job_id | Integer | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 转换结果 |

> 代码出处：`TemplateController.transTemplate`

#### 更新模板内容 `POST /v3/realtest/template/update_template_content`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realtest/template/update_template_content` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 更新模板内容 |

**请求参数**（`TemplateContentRequestDTO`）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| tagList | JSONArray | 否 | 标签列表 |
| jobIdList | JSONArray | 否 | 任务id列表 |
| editType | Integer | 否 | 编辑类型 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 更新结果 |

> 代码出处：`TemplateController.updateTemplateContent`

#### 列出模板 ID `GET /v3/realtest/template/list_template_id`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realtest/template/list_template_id` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 按项目/业务列出模板 id |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| project_id | Integer | 是 | 项目id |
| biz_code | Integer | 是 | 业务编码 |

**返回参数**（`data` = `TemplateIdDTO`）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.totalCount | Integer | 总数 |
| data.templateIds | JSONArray | 模板id列表 |

> 代码出处：`TemplateController.listTemplateId`

### 报告与导出 · Report（ApiServlet action=report，27 op）

#### list — 报告列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.list` |
| 鉴权 | needLogin=1 |
| 说明 | 分页查询报告列表（支持 skey 分享、历史/补测过滤） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| subtaskid | String | 否 | 子任务id |
| subsubtaskids | JSONArray | 否 | 子子任务id列表（元素 String） |
| keywords | JSONArray | 否 | 关键字列表（元素 String） |
| history | Boolean | 否 | 是否查询重试记录，默认 false |
| containRetest | Boolean | 否 | 是否查询补测记录，默认 false |
| page | Integer | 否 | 页码，默认 1 |
| pageSize | Integer | 否 | 每页大小，默认 200（1~200） |
| userid | Integer | 否 | 用户id |
| eid | Integer | 否 | 企业id |
| userprojectids | JSONArray | 否 | 用户项目组 ID 数组（元素 Integer） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 报告列表 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |

> 代码出处：`Report.list`

#### stepExtensionFile — 步骤扩展文件

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.stepExtensionFile` |
| 鉴权 | needLogin=1 |
| 说明 | 获取步骤扩展文件 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| subsubtaskid | String | 是 | 子子任务id |
| scriptTag | String | 是 | 脚本标签 |
| stepid | Integer | 是 | 步骤id |
| history | Boolean | 否 | 是否包含重试记录，默认 false |
| conditionKeys | JSONObject | 否 | 条件对象 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 扩展文件列表 |

> 代码出处：`Report.stepExtensionFile`

#### url — 报告地址

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.url` |
| 鉴权 | needLogin=1 |
| 说明 | 获取报告地址 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| projectid | Integer | 是 | 项目id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | String | 报告 url |

> 代码出处：`Report.url`

#### spots — 抽查信息

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.spots` |
| 鉴权 | needLogin=1 |
| 说明 | 获取抽查信息（分页） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（未做强校验） |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |
| round | Integer | 否 | 轮次 |
| item | String | 否 | 项目 |
| releaseVer | String | 否 | 发布版本 |
| dpiWidth | Integer | 否 | 屏幕宽 |
| dpiHeight | Integer | 否 | 屏幕高 |
| aliasName | String | 否 | 别名 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 抽查列表 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |

> 代码出处：`Report.spots`

#### conditions — 报告查询条件

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.conditions` |
| 鉴权 | needLogin=1 |
| 说明 | 获取报告查询条件 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id（与 skey 二选一，无则返回 infoMatch） |
| skey | String | 否 | 分享密钥 |
| resultCategorys | JSONArray | 否 | 结果分类列表（元素 Integer） |
| userid | Integer | 否 | 用户id |
| eid | Integer | 否 | 企业id |
| userprojectids | JSONArray | 否 | 用户项目组 ID 数组（元素 Integer） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 查询条件（map） |

> 代码出处：`Report.conditions`

#### listDeviceImages — 设备截图列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.listDeviceImages` |
| 鉴权 | needLogin=1 |
| 说明 | 设备截图列表（分页） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| errorMsg | String | 是 | 错误信息（CheckArgs 校验） |
| scriptNo | Integer | 是 | 脚本编号 |
| resultCategory | Integer | 是 | 结果分类 |
| scriptTags | String | 否 | 脚本标签 |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |
| userid | Integer | 否 | 用户id |
| eid | Integer | 否 | 企业id |
| userprojectids | JSONArray | 否 | 用户项目组 ID 数组 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 设备截图列表 |
| data.totalRow | Long | 总记录数 |
| data.totalPage | Integer | 总页数 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |

> 代码出处：`Report.listDeviceImages`

#### listDeviceReport — 设备报告列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.listDeviceReport` |
| 鉴权 | needLogin=1 |
| 说明 | 设备报告列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| round | Integer | 否 | 轮次（默认 1） |
| keyword | String | 否 | 关键字（设备名或错误定位） |
| status | JSONArray | 否 | 设备运行状态数组（-1 未通过/-2 待执行/-3 执行中/-4 失效） |
| resultCategorys | JSONArray | 否 | 结果分类数组 |
| brands | JSONArray | 否 | 品牌过滤（元素 String） |
| releaseVers | JSONArray | 否 | 版本过滤（元素 String） |
| resolutions | JSONArray | 否 | 分辨率过滤（元素 {"dpiWidth":Integer,"dpiHeight":Integer}） |
| userid | Integer | 否 | 用户id |
| eid | Integer | 否 | 企业id |
| userprojectids | JSONArray | 否 | 用户项目组 ID 数组 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 设备报告列表 |

> 代码出处：`Report.listDeviceReport`

#### adaptDetailRunInfoDetail — 适配详情运行信息

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.adaptDetailRunInfoDetail` |
| 鉴权 | needLogin=1 |
| 说明 | 获取适配详情运行信息 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| subtaskid | String | 否 | 子任务id |
| round | Integer | 否 | 轮次（默认 1） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | JSONObject | RealAdaptDetail 适配运行详情（代码未确认字段） |

> 代码出处：`Report.adaptDetailRunInfoDetail`

#### stepinfos — 步骤信息

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.stepinfos` |
| 鉴权 | needLogin=1 |
| 说明 | 获取步骤信息（按设备分组） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| scriptTag | String | 是 | 脚本标签 |
| subsubtaskidList | JSONArray | 是 | 子子任务id列表（元素 String，空抛 paraInvalid） |
| stepid | Integer | 是 | 步骤id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | Map<deviceId, StepInfo> |

> 代码出处：`Report.stepinfos`

#### scriptsummaries — 脚本摘要

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.scriptsummaries` |
| 鉴权 | needLogin=1 |
| 说明 | 获取脚本摘要列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| async | Integer | 否 | 是否异步 |
| orderNum | Integer | 否 | 序号 |
| callTag | String | 否 | 调用标签 |
| scriptid | Integer | 否 | 脚本id |
| userid | Integer | 否 | 用户id |
| eid | Integer | 否 | 企业id |
| userprojectids | JSONArray | 否 | 用户项目组 ID 数组 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 脚本摘要列表 |

> 代码出处：`Report.scriptsummaries`

#### scriptSteps — 脚本步骤

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.scriptSteps` |
| 鉴权 | needLogin=1 |
| 说明 | 获取脚本步骤信息 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| subtaskid | String | 是 | 子任务id |
| taskid | String | 是 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| findSubsubtaskid | String | 否 | 子子任务id |
| findRetryNum | Integer | 否 | 重试次数 |
| async | Integer | 否 | 是否异步 |
| orderNum | Integer | 否 | 序号 |
| callTag | String | 否 | 调用标签 |
| userid | Integer | 否 | 用户id |
| eid | Integer | 否 | 企业id |
| userprojectids | JSONArray | 否 | 用户项目组 ID 数组 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 步骤列表 |
| data.preResult | Boolean | 预处理结果 |
| data.processMap | JSONObject | 过程映射 |
| data.processArray | JSONArray | 过程数组 |

> 代码出处：`Report.scriptSteps`

#### stepdetail — 步骤详情

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.stepdetail` |
| 鉴权 | needLogin=1 |
| 说明 | 获取步骤详情 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| subsubtaskid | String | 是 | 子子任务id（CheckArgs 校验） |
| scriptTag | String | 是 | 脚本标签 |
| stepid | Integer | 是 | 步骤id |
| taskid | String | 否 | 任务id |
| skey | String | 否 | 分享密钥 |
| global | Integer | 否 | 是否全局 |
| userid | Integer | 否 | 用户id |
| eid | Integer | 否 | 企业id |
| userprojectids | JSONArray | 否 | 用户项目组 ID 数组 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | JSONObject | 步骤执行详情（Map<String,Object>，代码未确认字段） |

> 代码出处：`Report.stepdetail`

#### deviceInfo — 设备信息

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.deviceInfo` |
| 鉴权 | needLogin=1 |
| 说明 | 获取设备信息 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| subtaskid | String | 是 | 子任务id |
| taskid | String | 否 | 任务id |
| skey | String | 否 | 分享密钥 |
| orderNum | Integer | 否 | 序号 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | JSONArray | 设备信息列表 |

> 代码出处：`Report.deviceInfo`

#### stepinfoByLine — 按行号获取步骤信息

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.stepinfoByLine` |
| 鉴权 | needLogin=1 |
| 说明 | 按行号获取步骤信息 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| subtaskid | String | 是 | 子任务id |
| subsubtaskid | String | 是 | 子子任务id |
| line | Integer | 是 | 行号 |
| userid | Integer | 否 | 用户id |
| eid | Integer | 否 | 企业id |
| userprojectids | JSONArray | 否 | 用户项目组 ID 数组 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | StepInfo 步骤信息 |

> 代码出处：`Report.stepinfoByLine`

#### ignore — 忽略

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.ignore` |
| 鉴权 | needLogin=1 |
| 说明 | 忽略错误（CheckArgs 校验） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| subtaskids | JSONArray | 是 | 子任务id列表 |
| projectid | Integer | 是 | 项目id |
| subsubtaskids | JSONArray | 否 | 子子任务id列表 |
| round | Integer | 否 | 轮次 |
| userid | Integer | 否 | 用户id |
| username | String | 否 | 用户名 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Report.ignore`

#### modifyResultCategory — 修改结果分类

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.modifyResultCategory` |
| 鉴权 | needLogin=1 |
| 说明 | 修改结果分类 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| subtaskid | String | 是 | 子任务id |
| subsubtaskid | String | 是 | 子子任务id |
| projectid | Integer | 是 | 项目id |
| resultCategory | Integer | 是 | 结果分类 |
| round | Integer | 否 | 轮数，默认 1 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Report.modifyResultCategory`

#### deviceRestore — 设备恢复

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.deviceRestore` |
| 鉴权 | needLogin=1 |
| 说明 | 设备恢复 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| subtaskid | String | 是 | 子任务id |
| projectid | Integer | 是 | 项目id |
| round | Integer | 否 | 轮数，默认 1 |
| userid | Integer | 否 | 用户id |
| username | String | 否 | 用户名 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Report.deviceRestore`

#### scriptRestore — 脚本恢复

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.scriptRestore` |
| 鉴权 | needLogin=1 |
| 说明 | 脚本恢复 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| subtaskid | String | 是 | 子任务id |
| subsubtaskid | String | 是 | 子子任务id |
| projectid | Integer | 是 | 项目id |
| round | Integer | 否 | 轮数，默认 1 |
| userid | Integer | 否 | 用户id |
| username | String | 否 | 用户名 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Report.scriptRestore`

#### testProcess — 测试过程

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.testProcess` |
| 鉴权 | needLogin=1 |
| 说明 | 获取测试过程 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（与 subsubtaskid 二选一） |
| subsubtaskid | String | 否 | 子子任务id |
| name | String | 否 | 名称 |
| stage | String | 否 | 阶段 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 测试过程列表 |

> 代码出处：`Report.testProcess`

#### adaptResults — 适配结果

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.adaptResults` |
| 鉴权 | needLogin=1 |
| 说明 | 获取适配结果 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| subtaskid | String | 否 | 子任务id |
| round | Integer | 否 | 轮次 |
| containRetest | Integer | 否 | 是否包含补测 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 适配结果列表 |

> 代码出处：`Report.adaptResults`

#### getStatSummary — 统计摘要

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.getStatSummary` |
| 鉴权 | needLogin=1 |
| 说明 | 获取统计摘要（分页） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| page | Integer | 是 | 当前页 |
| pageSize | Integer | 是 | 每页大小 |
| keywords | JSONArray | 否 | 关键字 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | PmrealStatSummary 统计摘要 |

> 代码出处：`Report.getStatSummary`

#### getErrorSummary — 错误摘要

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.getErrorSummary` |
| 鉴权 | needLogin=1 |
| 说明 | 获取错误摘要 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id |
| subtaskid | String | 否 | 子任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | 错误摘要（TestErrorSummary，查询为空时无此字段） |

> 代码出处：`Report.getErrorSummary`

#### singleScriptSummary — 单脚本摘要

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.singleScriptSummary` |
| 鉴权 | needLogin=1 |
| 说明 | 获取单脚本摘要 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| scriptNo | Integer | 是 | 脚本编号 |
| scriptid | Integer | 是 | 脚本id |
| orderNum | Integer | 是 | 序号 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | 脚本摘要对象 |

> 代码出处：`Report.singleScriptSummary`

#### getRealRecordDetail — 获取真机记录详情

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.getRealRecordDetail` |
| 鉴权 | needLogin=1 |
| 说明 | 获取真机记录详情（skey 或 taskid 查询） |

**请求参数**（RealRecordDetailDTO）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| skey | String | 否 | 分享密钥（与 taskid 二选一） |
| taskid | String | 否 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | RealRecordDetail 详情 |

> 代码出处：`Report.getRealRecordDetail`

#### saveRealRecord — 保存真机记录

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.saveRealRecord` |
| 鉴权 | needLogin=1 |
| 说明 | 保存真机记录 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| deviceArray | JSONArray | 否 | 设备数据（getDataFromDeviceArray 读取，无显式字段校验） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |

> 代码出处：`Report.saveRealRecord`

#### getRealRecordList — 获取真机记录列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.getRealRecordList` |
| 鉴权 | needLogin=1 |
| 说明 | 分页获取真机记录列表 |

**请求参数**（RealRecordListRequestDTO）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| page | Integer | 是 | 当前页 |
| pageSize | Integer | 是 | 每页大小 |
| eid | Integer | 否 | 企业id |
| projectId | Integer | 否 | 项目id |
| userId | Integer | 否 | 用户id |
| userName | String | 否 | 用户名 |
| startTime | Long | 否 | 开始时间 |
| endTime | Long | 否 | 结束时间 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 记录列表 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Long | 总页数 |
| data.totalRow | Long | 总行数 |

> 代码出处：`Report.getRealRecordList`

#### getImages — 获取截图

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Report.getImages` |
| 鉴权 | needLogin=1 |
| 说明 | 获取截图列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| type | String | 是 | 类型 |
| pkgName | String | 否 | 包名 |
| times | Integer | 否 | 次数 |
| time | String | 否 | 时间 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | JSONArray | 截图列表 |

> 代码出处：`Report.getImages`

### 报告与导出 · Excel（ApiServlet action=report，3 op）

#### create — 创建 Excel

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Excel.create` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 创建 Excel 报告 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Excel.create`

#### generate — 生成 Excel

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Excel.generate` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 生成 Excel 报告 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Excel.generate`

#### reportExcel — Excel 导出

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Excel.reportExcel` |
| 鉴权 | needLogin=1 |
| 说明 | Excel 报告导出 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| eid | Integer | 否 | 企业id |
| userid | Integer | 否 | 用户id |
| userprojectids | JSONArray | 否 | 用户项目组 ID 数组（元素 Integer） |
| lastUpdatetime | Long | 否 | 最后更新时间 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 状态（1 成功 / 0 生成中 / -1 生成失败 / -2 待执行） |
| data.excelUrl | String | Excel 地址 |

> 代码出处：`Excel.reportExcel`

### 报告与导出 · Pdf（ApiServlet action=report，1 op）

#### parse — PDF 报告解析生成

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Pdf.parse` |
| 鉴权 | needLogin=1 |
| 说明 | PDF 报告解析生成 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| url | String | 是 | 报告 url |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | ResponseBean（code/md5Key/targetUrl） |

> 代码出处：`Pdf.parse`

### 报告与导出 · Qc（ApiServlet action=report，2 op）

#### notify — 质检通知

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Qc.notify` |
| 鉴权 | needLogin=1 |
| 说明 | 质检通知 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业id |
| projectid | Integer | 否 | 项目id |
| taskid | String | 否 | 任务id |
| subtaskid | String | 否 | 子任务id |
| subsubtaskid | String | 否 | 子子任务id |
| title | String | 否 | 标题 |
| descr | String | 否 | 描述 |
| scriptNo | Integer | 否 | 脚本编号 |
| scriptDescr | String | 否 | 脚本描述 |
| scriptTags | String | 否 | 脚本标签 |
| stepid | Integer | 否 | 步骤id |
| userid | Integer | 否 | 用户id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Qc.notify`

#### syncInfo — 质检信息同步

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Qc.syncInfo` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 质检信息同步 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业id |
| projectid | Integer | 否 | 项目id |
| taskid | String | 否 | 任务id |
| subtaskid | String | 否 | 子任务id |
| subsubtaskid | String | 否 | 子子任务id |
| qcid | String | 否 | 质检id |
| userid | Integer | 否 | 用户id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Qc.syncInfo`

### 报告与导出 · ScriptReport（ApiServlet action=report，1 op）

#### checkInfos — 脚本报告检查信息

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.ScriptReport.checkInfos` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 脚本报告检查信息（分页） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| subtaskid | String | 否 | 子任务id |
| subsubtaskid | String | 否 | 子子任务id |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 检查信息列表 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |

> 代码出处：`ScriptReport.checkInfos`

### 报告与导出 · Stat（ApiServlet action=report，1 op）

#### exceptions — 异常统计分析

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.Stat.exceptions` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 异常统计分析（分页） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| type | Integer | 否 | 类型 |
| subtaskid | String | 否 | 子任务id |
| deviceid | String | 否 | 设备id |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 异常列表 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |

> 代码出处：`Stat.exceptions`

---

## 三、性能与问题分析（20）

### 性能与问题分析 · PerformanceController（WebMvc `/realtest`，3 接口）

#### 性能数据查询 `GET /v3/realtest/report/performance`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realtest/report/performance` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 查询性能数据 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | JSONArray | 性能数据列表（ReportPerformanceResponse） |

> 代码出处：`PerformanceController.performance`

#### 抽查信息 `POST /v3/realtest/report/spot`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realtest/report/spot` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 测试计划记录抽查信息 |

**请求参数**（`SpotRequestDTO`）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskIds | JSONArray | 是 | 任务id列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | JSONArray | 抽查信息列表（SpotInformation） |

> 代码出处：`PerformanceController.testPlanRecordSpot`

#### 刷新执行概要 `POST /v3/realtest/report/refresh_report_input_param`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realtest/report/refresh_report_input_param` |
| 鉴权 | needLogin=1 |
| 说明 | 刷新报告输入参数（执行概要） |

**请求参数**（`UpdateReportInputParamDTO`，继承 `BaseQueryRequestDTO`）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业id |
| projectId | Integer | 否 | 项目id |
| userId | Integer | 否 | 用户id |
| userName | String | 否 | 用户名 |
| taskId | String | 否 | 任务id |
| subTaskId | String | 否 | 子任务id |
| subSubTaskId | String | 否 | 子子任务id |
| taskIds | JSONArray | 否 | 任务id列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 刷新结果 |

> 代码出处：`PerformanceController.updateReportInputParam`

### 性能与问题分析 · ProblemAnalysisReportController（WebMvc `/realtest`，3 接口）

#### 问题分析列表 `POST /v3/realtest/report/script_list`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realtest/report/script_list` |
| 鉴权 | needLogin=1 |
| 说明 | 脚本问题分析列表（分页） |

**请求参数**（`ReportScriptListRequestDTO`）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskId | String | 是 | 任务id |
| projectId | Integer | 否 | 项目id |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |
| scriptNo | Integer | 否 | 脚本编号 |
| scriptName | String | 否 | 脚本名称 |
| testResult | Integer | 否 | 测试结果 |
| errorCauseTypeId | Integer | 否 | 错误类型id |
| customizeErrorMsg | String | 否 | 自定义错误信息 |
| deviceId | String | 否 | 设备id |
| systemError | String | 否 | 系统错误 |
| isAnalyze | Integer | 否 | 是否分析 |
| resultCategory | Integer | 否 | 结果分类 |
| deviceIp | String | 否 | 设备ip |
| sortFields | JSONArray | 否 | 排序字段 |

**返回参数**（`data` = `PageResponseDTO<ScriptProblemAnalysisDTO>`）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |
| data.list | JSONArray | 问题分析列表（ScriptProblemAnalysisDTO） |

> 代码出处：`ProblemAnalysisReportController.getReportScriptList`

#### 错误步骤详情 `GET /v3/realtest/report/error_step_report_detail_list`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realtest/report/error_step_report_detail_list` |
| 鉴权 | needLogin=1 |
| 说明 | 错误步骤报告详情列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 任务id |
| subsubtask_id | String | 是 | 子子任务id |
| size | Integer | 否 | 返回条数，默认 5 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 错误步骤详情列表（Map<String,Object>） |
| data.totalRow | Integer | 总记录数（=list 大小） |

> 代码出处：`ProblemAnalysisReportController.getErrorScriptReportDetailList`

#### 问题分析设备列表 `GET /v3/realtest/report/device_list/{task_id}`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realtest/report/device_list/{task_id}`（路由表占位符为 `{param1}`） |
| 鉴权 | needLogin=1 |
| 说明 | 问题分析设备列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 路径变量，任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 设备列表（Map<String,Object>） |

> 代码出处：`ProblemAnalysisReportController.getDeviceList`

### 性能与问题分析 · Performance（ApiServlet action=analysis，2 op）

#### reportGraph — 性能图表数据

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `analysis.Performance.reportGraph` |
| 鉴权 | needLogin=1 |
| 说明 | 性能图表数据 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| subtaskid | String | 是 | 子任务id |
| taskid | String | 否 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| history | Boolean | 否 | 是否查询重试记录，默认 false |
| containRetest | Boolean | 否 | 是否查询补测记录，默认 false |
| subsubtaskid | JSONArray | 否 | 子子任务id数组（元素 String） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 性能图表数据 map |

> 代码出处：`Performance.reportGraph`

#### performanceExport — 性能数据导出

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `analysis.Performance.performanceExport` |
| 鉴权 | needLogin=1 |
| 说明 | 性能数据导出 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| subtaskid | String | 是 | 子任务id |
| taskid | String | 否 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| history | Boolean | 否 | 是否查询重试记录，默认 false |
| containRetest | Boolean | 否 | 是否查询补测记录，默认 false |
| subsubtaskid | JSONArray | 否 | 子子任务id数组（元素 String） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | String | 导出 url（或 0） |

> 代码出处：`Performance.performanceExport`

### 性能与问题分析 · AnalysisReport（ApiServlet action=analysis，4 op）

#### list — 报告分析列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `analysis.Report.list` |
| 鉴权 | needLogin=1 |
| 说明 | 报告分析列表（分页） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| subtaskid | String | 否 | 子任务id |
| subtaskids | JSONArray | 否 | 子任务id列表 |
| subsubtaskid | String | 否 | 子子任务id |
| subSubTaskIds | JSONArray | 否 | 子子任务id列表 |
| keyword | String | 否 | 关键字 |
| resultCategorys | JSONArray | 否 | 结果分类 |
| scriptTags | JSONArray | 否 | 脚本标签 |
| scriptNos | JSONArray | 否 | 脚本编号 |
| scriptDescrs | JSONArray | 否 | 脚本描述 |
| deviceNames | JSONArray | 否 | 设备名 |
| errorMsgs | JSONArray | 否 | 错误信息 |
| timeConsumings | JSONArray | 否 | 耗时 |
| retryNum | Integer | 否 | 重试次数 |
| hasSupplementary | Integer | 否 | 是否有补充 |
| sortFields | JSONArray | 否 | 排序字段 |
| keywords | JSONArray | 否 | 关键字列表 |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 分析列表 |

> 代码出处：`AnalysisReport.list`

#### summaries — 报告分析汇总

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `analysis.Report.summaries` |
| 鉴权 | needLogin=1 |
| 说明 | 报告分析汇总 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| type | String | 否 | 类型 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 汇总数据 |

> 代码出处：`AnalysisReport.summaries`

#### issues — 问题列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `analysis.Report.issues` |
| 鉴权 | needLogin=1 |
| 说明 | 问题列表（分页） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sid | String | 是 | 会话/分享标识 |
| taskid | String | 是 | 任务id |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.rows | JSONArray | 问题行 |
| data.total | Integer | 总数 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |

> 代码出处：`AnalysisReport.issues`

#### modifyErrorMsg — 修改错误信息

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `analysis.Report.modifyErrorMsg` |
| 鉴权 | needLogin=1 |
| 说明 | 修改错误信息 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| subtaskid | String | 是 | 子任务id |
| subsubtaskid | String | 是 | 子子任务id |
| errorMsg | String | 否 | 错误信息 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 修改结果（0/1） |

> 代码出处：`AnalysisReport.modifyErrorMsg`

### 性能与问题分析 · AnalysisTask（ApiServlet action=analysis，4 op）

#### overview — 任务分析概况

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `analysis.Task.overview` |
| 鉴权 | needLogin=1 |
| 说明 | 任务分析概况 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id（与 skey 二选一） |
| skey | String | 否 | 分享密钥 |
| keywords | JSONArray | 否 | 关键字 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | PmrealTaskSummary 任务概况 |

> 代码出处：`AnalysisTask.overview`

#### subSummarys — 子任务摘要（@Deprecated）

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `analysis.Task.subSummarys` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 子任务摘要（已废弃，分页） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 子任务摘要列表 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |

> 代码出处：`AnalysisTask.subSummarys`

#### performanceSummary — 性能摘要（@Deprecated）

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `analysis.Task.performanceSummary` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 性能摘要（已废弃） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | 性能摘要对象 |

> 代码出处：`AnalysisTask.performanceSummary`

#### getDeviceExecStatus — 设备执行状态（@Deprecated）

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `analysis.Task.getDeviceExecStatus` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 设备执行状态（已废弃） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | 设备执行状态对象 |

> 代码出处：`AnalysisTask.getDeviceExecStatus`

### 性能与问题分析 · Safety（ApiServlet action=app，4 op，@Deprecated）

#### insertSafeInfo — 新增安全检测信息

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Safety.insertSafeInfo` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 新增加固安全检测信息（已废弃） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| projectId | Integer | 是 | 项目id |
| taskId | String | 是 | 任务id |
| subTaskId | String | 是 | 子任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 插入记录 ID |

> 代码出处：`Safety.insertSafeInfo`

#### updateSafeInfo — 更新安全检测信息

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Safety.updateSafeInfo` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 更新安全检测信息（已废弃） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Integer | 是 | 记录id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 更新记录 ID |

> 代码出处：`Safety.updateSafeInfo`

#### selectSafeInfo — 查询安全检测信息

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Safety.selectSafeInfo` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 查询单条安全检测信息（已废弃） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Integer | 是 | 记录id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | JSONObject | 安全检测详情（BangcleSafety） |

> 代码出处：`Safety.selectSafeInfo`

#### listSafeInfo — 安全检测信息列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Safety.listSafeInfo` |
| 鉴权 | needLogin=1 |
| 说明 | 安全检测信息列表（无参数校验） |

**请求参数**：无必填字段（无显式校验）。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 安全检测信息列表 |

> 代码出处：`Safety.listSafeInfo`

---

## 四、测试资源（16）

### 测试资源 · ScriptList（ApiServlet action=script，7 op）

#### getAppVersions — 获取应用版本列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `script.ScriptList.getAppVersions` |
| 鉴权 | needLogin=1 |
| 说明 | 获取应用版本列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| appId | Integer | 否 | 应用id |
| suiteId | Integer | 否 | Suite id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | JSONArray | 版本列表 |

> 代码出处：`ScriptList.getAppVersions`

#### projectSuiteListInfo — 项目 Suite 列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `script.ScriptList.projectSuiteListInfo` |
| 鉴权 | needLogin=1 |
| 说明 | 获取项目 Suite 列表信息 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | PageList 分页列表 |

> 代码出处：`ScriptList.projectSuiteListInfo`

#### projectSuiteApps — 项目 Suite 应用列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `script.ScriptList.projectSuiteApps` |
| 鉴权 | needLogin=1 |
| 说明 | 获取项目 Suite 应用列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| suiteId | Integer | 是 | Suite id |
| syspfId | Integer | 否 | 系统平台id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | JSONArray | 应用列表 |

> 代码出处：`ScriptList.projectSuiteApps`

#### collectionsScriptList — 收藏夹脚本列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `script.ScriptList.collectionsScriptList` |
| 鉴权 | needLogin=1 |
| 说明 | 获取收藏夹脚本列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| scriptDirId | Integer | 否 | 脚本目录id |
| deep | Integer | 否 | 深度 |
| eid | Integer | 否 | 企业id |
| projectid | Integer | 否 | 项目id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | JSONArray | scriptNoList 脚本编号列表 |

> 代码出处：`ScriptList.collectionsScriptList`

#### listScript — 脚本列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `script.ScriptList.listScript` |
| 鉴权 | needLogin=1 |
| 说明 | 条件查询脚本列表 |

**请求参数**（多个可选过滤条件，代码未逐字段确认必填）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业id |
| projectid | Integer | 否 | 项目id |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |
| keyword | String | 否 | 关键字 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 脚本列表 |
| data.pager | JSONObject | 分页信息 |
| data.todayList | JSONArray | 今日列表 |
| data.yesterdayList | JSONArray | 昨日列表 |
| data.beforeList | JSONArray | 更早列表 |
| data.counts | JSONObject | 统计计数 |

> 代码出处：`ScriptList.listScript`

#### collectionsList — 收藏夹列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `script.ScriptList.collectionsList` |
| 鉴权 | needLogin=1 |
| 说明 | 获取收藏夹列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| lazyTree | Integer | 是 | 懒加载树 |
| parentDirId | Integer | 是 | 父目录id |
| projectid | Integer | 是 | 项目id |
| dirType | Integer | 是 | 目录类型 |
| eid | Integer | 是 | 企业id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | JSONArray | 收藏夹列表 |

> 代码出处：`ScriptList.collectionsList`

#### scriptGroupList — 脚本组列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `script.ScriptList.scriptGroupList` |
| 鉴权 | needLogin=1 |
| 说明 | 获取脚本组列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| appid | Integer | 否 | 应用id |
| suiteId | Integer | 否 | Suite id |
| keywordType | Integer | 否 | 关键字类型 |
| keyword | String | 否 | 关键字 |
| pageNo | Integer | 否 | 页码 |
| scriptType | Integer | 否 | 脚本类型 |
| pageSize | Integer | 否 | 每页大小 |
| projectid | Integer | 否 | 项目id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | JSONArray | 脚本组列表（NewScriptGroup） |
| data.pageInfo | JSONObject | 分页信息 |

> 代码出处：`ScriptList.scriptGroupList`

### 测试资源 · ParamSource（ApiServlet action=app，1 op）

#### assign — 根据设备分配数据源

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.ParamSource.assign` |
| 鉴权 | needLogin=0 |
| 说明 | 根据设备分配数据源 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| paramSource | String | 是 | 参数来源 |
| devices | JSONArray | 是 | 设备列表 |
| scripts | JSONArray | 是 | 脚本列表 |
| type | Integer | 否 | 类型 |
| globalParamIndex | Integer | 否 | 全局参数索引 |
| tagList | JSONArray | 否 | 标签列表 |
| noHasTagList | JSONArray | 否 | 无标签列表 |
| oldTaskId | String | 否 | 旧任务id |
| scriptStatus | JSONArray | 否 | 脚本状态 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 分配结果列表 |

> 代码出处：`ParamSource.assign`

### 测试资源 · Account（ApiServlet action=app，2 op）

#### match — 测试账号匹配

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Account.match` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 测试账号匹配 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id |
| subtaskid | String | 否 | 子任务id |
| deviceid | String | 否 | 设备id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | adaptAccount 适配账号 |

> 代码出处：`Account.match`

#### release — 测试账号释放

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `app.Account.release` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 测试账号释放 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 否 | 任务id |
| subtaskid | String | 否 | 子任务id |
| deviceid | String | 否 | 设备id |
| accountId | String | 否 | 账号id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`Account.release`

### 测试资源 · SpotReference（ApiServlet action=report，2 op）

#### maintain — 抽查参考信息维护

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.SpotReference.maintain` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 维护抽查参考信息 |

**请求参数**（RealSpotReference.toBean 校验）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目id |
| appid | Integer | 是 | 应用id |
| name | String | 是 | 名称 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`SpotReference.maintain`

#### list — 抽查参考信息列表

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `report.SpotReference.list` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 抽查参考信息列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| name | String | 否 | 名称 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 抽查参考列表 |

> 代码出处：`SpotReference.list`

### 测试资源 · FreeQuota（ApiServlet action=free，4 op，@Deprecated）

#### addQuota — 新增免费额度

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `free.FreeQuota.addQuota` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 新增免费额度（已废弃） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| bizCode | Integer | 否 | 业务编码 |
| eid | Integer | 否 | 企业id |
| pid | Integer | 否 | 项目id |
| totalNum | Integer | 否 | 总数量 |
| num | Integer | 否 | 数量 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 新增结果 |

> 代码出处：`FreeQuota.addQuota`

#### modifyQuota — 修改免费额度

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `free.FreeQuota.modifyQuota` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 修改免费额度（已废弃） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| bizCode | Integer | 否 | 业务编码 |
| eid | Integer | 否 | 企业id |
| pid | Integer | 否 | 项目id |
| totalNum | Integer | 否 | 总数量 |
| num | Integer | 否 | 数量 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 修改结果 |

> 代码出处：`FreeQuota.modifyQuota`

#### get — 查询免费额度

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `free.FreeQuota.get` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 查询免费额度（已废弃） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| bizCode | Integer | 是 | 业务编码 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data.objInfo | JSONObject | 额度对象 |

> 代码出处：`FreeQuota.get`

#### deduction — 扣减免费额度

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST |
| 路径 | `free.FreeQuota.deduction` |
| 鉴权 | needLogin=1（代码未确认） |
| 说明 | 扣减免费额度（已废弃） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| bizCode | Integer | 是 | 业务编码 |
| num | Integer | 是 | 扣减数量 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码：0 成功 / 10003 配额不足 / 10000 未知错误 |
| msg | String | 提示信息 |

> 代码出处：`FreeQuota.deduction`

---

## 五、基础设施（1）

### 基础设施 · HeartBeatController（WebMvc `/realtest`，1 接口）

#### 健康检查 `GET /v3/realtest/heartbeat/check`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realtest/heartbeat/check` |
| 鉴权 | needLogin=0 |
| 说明 | MySQL + MongoDB + Redis + ES 健康检查，任一异常抛 HealthyCheckGenericException |

**请求参数**：无。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码 0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 空对象（HashMap） |

> 代码出处：`HeartBeatController.check`

---

## 附：路由表 V3→V1 转换接口（2）

以下为网关在 V3 层暴露的 URL，实际转换为 V1 ApiServlet op 执行（passThroughType=0 + special_api_action/op）。

### 分享报告生成 skey `GET /v3/realtest/task/share`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟡 |
| HTTP 方法 | GET |
| 路径 | `/v3/realtest/task/share` |
| 鉴权 | needLogin=1 |
| 说明 | 转为 V1 `app.Task.share`，分享报告生成 skey |

**请求参数**：同 `Task.share`（taskid/userid/eid 等）。

**返回参数**：同 `Task.share`（data.result = skey）。

> 代码出处：`Task.share`（网关 V3→V1 转换）

### 取消某设备子任务 `PUT /v3/realtest/task/stop_task`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟡 |
| HTTP 方法 | PUT |
| 路径 | `/v3/realtest/task/stop_task` |
| 鉴权 | needLogin=1 |
| 说明 | 转为 V1 `app.Task.cancel`，取消某设备子任务 |

**请求参数**：同 `Task.cancel`（taskid/subtaskid 等）。

**返回参数**：同 `Task.cancel`（data.result = 1/0）。

> 代码出处：`Task.cancel`（网关 V3→V1 转换）

---

## 附：代码未确认/异常清单

| 接口 | 问题 | 说明 |
|---|---|---|
| `GET /v3/realtest/report/steps`（🔵） | 代码未确认 | 路由表标注指向 ReportController，但该 Controller 无 `/steps` 端点，代码中未找到对应映射 |
| 部分 V3 端点 needLogin | 代码未确认 | 路由表未收录的 Controller 端点，needLogin 按同模块默认 1 标注 |
| `ReTestInfoDTO` / `EsReportSummary` / `SubSubTaskInfoResponseDTO` / `ScriptProblemAnalysisDTO` 等 DTO 明细字段 | 代码未确认 | 未在本次梳理中逐一展开字段定义 |
