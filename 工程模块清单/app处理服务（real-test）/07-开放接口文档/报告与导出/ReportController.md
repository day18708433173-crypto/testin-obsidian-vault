---
branch: syy.release.z7.8.1.0
module: real-test
type: 接口文档
entry: WebMvc
---

# ReportController

报告管理的 MVC 控制器，提供补测历史查询、报告列表查询、报告汇总、错误类型修改等功能。

类路径：`real-test/src/main/java/cn/testin/mvc/controller/ReportController.java`，基础路径 `/v3/report`。

## 本类接口一览

| 接口 | 方法 | 路径 | 功能 |
| --- | --- | --- | --- |
| list (reTestInfo) | GET | /v3/report/report/{task_id} | 分页查询补测历史记录 |
| list (reportList) | POST | /v3/report/report/list | 根据 taskIds 批量查询 ES 报告摘要 |
| summary | POST | /v3/report/report/summary | 根据条件查询报告汇总数据 |
| modifyErrorCauseType | POST | /v3/report/report/modify_error_cause_type | 批量修改报告错误原因类型 |

## list -- 补测记录 (`GET /v3/report/report/{task_id}`)

- **实现意图**：分页查询指定任务子任务的补测历史记录。

- **请求参数**：

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| task_id | String | 是 | 任务 ID（路径） |
| page | Integer | 是 | 页码 |
| page_size | Integer | 是 | 每页条数 |
| sub_sub_task_id | String | 是 | 子子任务 ID |
| start_time | Long | 否 | 开始时间戳 |
| end_time | Long | 否 | 结束时间戳 |

- **返回参数**：`ResponseResult<PageResponseDTO<ReTestInfoDTO>>`。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页条数 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总记录数 |
| data.list | JSONArray | 补测记录列表，元素为 `ReTestInfoDTO` |
| data.list[].scriptNo | Integer | 脚本编号 |
| data.list[].scriptName | String | 脚本名称 |
| data.list[].scriptTags | JSONArray | 脚本标签（元素 String） |
| data.list[].appReportDevice | JSONObject | 设备信息（`ReportDevice`，含 cloud/deviceid/modelName/brandName/execStatus 等） |
| data.list[].execStatus | Integer | 执行状态 |
| data.list[].timeConsuming | Long | 耗时 |
| data.list[].errorMsg | String | 错误定位 |
| data.list[].errorCode | Integer | 错误代码 |
| data.list[].taskId | String | 任务 ID |
| data.list[].subTaskId | String | 子任务 ID |
| data.list[].subSubTaskId | String | 子子任务 ID |
| data.list[].outputParams | String | 执行概要 |
| data.list[].inputParams | String | 输入变量信息 |
| data.list[].resultCategory | Integer | 状态 |
| data.list[].reportRunInfo | JSONObject | 运行信息（`ReportRunInfo`，含 ucomid/errorCode/errorMsg/execTime 等） |
| data.list[].sourceConfigId | Integer | 当前脚本使用的实例表 ID |
| data.list[].sourceConfigName | String | 当前脚本使用的实例表名称 |
| data.list[].sourceConfigParentId | Integer | 当前脚本使用的实例表的父文件夹 ID |

- **调用链**：[ReportService](ReportService.md).reTestInfo -> ES 查询补测记录。
- **涉及表与 SQL**：Elasticsearch 索引（报告数据）。

## list -- 报告列表 (`POST /v3/report/report/list`)

- **实现意图**：根据 `ReportListRequestDTO` 中的 taskIds 批量查询 ES 中的报告摘要。

- **请求参数**：body `ReportListRequestDTO`。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| projectId | Integer | 否 | 项目 ID |
| taskIds | JSONArray | 否 | 任务 ID 列表（元素 String） |
| subSubTaskIds | JSONArray | 否 | 子子任务 ID 列表（元素 String） |

- **返回参数**：`ResponseResult<ResultListResponseDTO<EsReportSummary>>`。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 报告摘要列表，元素为 `EsReportSummary` |
| data.list[].id | String | ES 文档 ID |
| data.list[].matchtime | Long | 任务匹配时间 |
| data.list[].taskid | String | 任务 ID |
| data.list[].subtaskid | String | 子任务 ID |
| data.list[].subsubtaskid | String | 子子任务 ID |
| data.list[].errorCode | Integer | 上位机错误对照码 |
| data.list[].pferrorCode | Integer | 平台 errorCode |
| data.list[].errorMsg | String | 错误信息 |
| data.list[].reportScript | JSONObject | 脚本相关（`ReportScript`） |
| data.list[].retryNum | Integer | 是否为重测 |
| data.list[].hasSupplementary | Integer | 是否为补测（0 否 1 是） |
| data.list[].imgUrl | String | 对应步骤截图 |
| data.list[].reportLogExceptions | JSONArray | log 结果错误信息（元素 `ReportLogException`） |
| data.list[].logresUrl | String | log 结果地址 |
| data.list[].originLogUrl | String | 原始 log 地址 |
| data.list[].reportDevice | JSONObject | 设备信息（`ReportDevice`） |
| data.list[].timeConsuming | Long | 耗时 |
| data.list[].resultCategory | Integer | 状态 |
| data.list[].reportRunInfo | JSONObject | runinfo（`ReportRunInfo`） |
| data.list[].scriptMark | String | 错误步骤对应关系 |
| data.list[].qcnotify | Integer | qc 通知模块 |
| data.list[].qcid | String | 同步 qc 的 id |
| data.list[].ignoreMark | Integer | 脚本集忽略标记（1 忽略 0 未忽略） |
| data.list[].originResultCategory | Integer | 历史结果信息 |
| data.list[].outputParams | String | 子子任务运行后全局变量信息 |
| data.list[].inputParams | String | 子子任务运行前全局变量信息 |
| data.list[].originalInputParams | String | 脚本执行前原始输入参数信息 |
| data.list[].orderNum | Integer | 排序号 |
| data.list[].dataRow | Integer | 数据行号 |
| data.list[].scriptDataRow | JSONObject | 脚本及子脚本使用的数据表行数（Map<Integer,Integer>） |
| data.list[].repeatTestMark | Integer | 补测标记（0 正常 1 补测） |
| data.list[].ciccDyeUrl | String | 中金代码染色报告地址 |
| data.list[].pmrealAdaptAppInfo | JSONObject | 适配应用信息（`PmrealAdaptAppInfo`） |
| data.list[].standard | String | 脚本执行策略 |
| data.list[].sourceConfigId | Integer | 当前脚本使用的实例表 ID |
| data.list[].sourceConfigName | String | 当前脚本使用的实例表名称 |
| data.list[].sourceConfigParentId | Integer | 当前脚本使用的实例表的父文件夹 ID |
| data.list[].customFileUrl | String | 用户 App 产生的日志文件 |
| data.list[].reportExecuteStatus | Integer | 报告执行状态 |
| data.list[].customizeErrorMsg | String | 自定义错误信息 |
| data.list[].handleInPutParamSign | Integer | 输入参数处理标记 |
| data.list[].precompleteTime | Long | 脚本执行完成时间 |
| data.list[].errorCauseTypeId | Integer | 错误原因类型 ID |

- **调用链**：[ReportService](ReportService.md).reportList -> `IEsReportSummaryDAO` -> Elasticsearch。
- **涉及表与 SQL**：Elasticsearch `report_summary` 索引。

## summary (`POST /v3/report/report/summary`)

- **实现意图**：根据条件查询报告汇总（同 list 但可能包含不同聚合维度）。

- **请求参数**：body `ReportListRequestDTO`（同 report/list）。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| projectId | Integer | 否 | 项目 ID |
| taskIds | JSONArray | 否 | 任务 ID 列表（元素 String） |
| subSubTaskIds | JSONArray | 否 | 子子任务 ID 列表（元素 String） |

- **返回参数**：`ResponseResult<ResultListResponseDTO<EsReportSummary>>`，data.list 元素字段同 `report/list`。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 报告汇总列表，元素为 `EsReportSummary`（字段同上） |

- **调用链**：[ReportService](ReportService.md).reportSummary -> `IEsReportSummaryDAO`。

## modifyErrorCauseType (`POST /v3/report/report/modify_error_cause_type`)

- **实现意图**：批量修改报告的错误原因类型（用于错误归因维护）。

- **请求参数**：body `MaintainErrorCauseTypeDTO`（继承 `BaseQueryRequestDTO`）。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| errorCauseTypeId | Integer | 否 | 错误原因类型 ID |
| taskId | String | 否 | 任务 ID |
| subTaskId | String | 否 | 子任务 ID |
| subSubTaskId | String | 否 | 子子任务 ID |
| eid | Integer | 否 | 企业 ID |
| projectId | Integer | 否 | 项目 ID |
| userId | Integer | 否 | 用户 ID |
| userName | String | 否 | 用户名 |

- **返回参数**：`ResponseResult<BaseResponseDTO>`。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 操作影响行数 |

- **调用链**：[ReportService](ReportService.md).modifyErrorCauseType -> DAO 更新错误类型。
