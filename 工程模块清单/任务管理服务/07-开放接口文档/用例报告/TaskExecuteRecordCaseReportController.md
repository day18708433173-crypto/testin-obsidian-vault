# TaskExecuteRecordCaseReportController — 用例报告详情

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/task/TaskExecuteRecordCaseReportController.java`
> 类级路由：`/real_task/case_report`
> Service 实现：`cn.testin.service.impl.task.TaskExecuteRecordServiceImpl`（部分方法）
> 业务：用例报告维度的详情与失败用例图表数据。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 1 | GET | `/v3/real_task/case_report/detail` | reportDetail | 报告详情（含脚本/用例/设备汇总） |
| 2 | GET | `/v3/real_task/case_report/fail_case_detail` | failCaseDetail | 失败用例图表信息 |

---

## 1. GET /v3/real_task/case_report/detail — 报告详情

### 入口

`TaskExecuteRecordCaseReportController.reportDetail(@RequestParam task_execute_record_id)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_execute_record_id | Integer | 是 | 执行记录id（@RequestParam） |

### 响应结构

`ResponseResult<TaskExecuteReportResponse>`，含：
- 任务基本信息（名称、状态、时间）
- 脚本/用例执行统计（总数、成功、失败）
- 设备使用统计
- 各脚本/用例执行结果列表

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.executeReportDesc | JSONObject | 报告基本字段（ExecuteReportDesc） |
| data.executeReportDesc.suiteId | Integer | 应用id |
| data.executeReportDesc.suiteName | String | 应用名 |
| data.executeReportDesc.suiteDesc | String | 备注 |
| data.executeReportDesc.pkgName | String | 包名 |
| data.executeReportDesc.versionRemark | String | 包备注 |
| data.executeReportDesc.appIconUrl | String | 应用图标 |
| data.executeReportDesc.appBuild | String | app 版本号 |
| data.executeReportDesc.versionName | String | app 版本名称 |
| data.executeReportDesc.appName | String | app 名称 |
| data.executeReportDesc.systemPlatformId | Integer | 包关联的系统平台id |
| data.executeReportDesc.createUserId | Integer | 创建人 |
| data.executeReportDesc.createUserEmail | String | 创建人邮箱 |
| data.executeReportDesc.executeMethod | Integer | 执行方式 |
| data.executeReportDesc.dataSourceId | Integer | 数据源 |
| data.executeReportDesc.dataSourceName | String | 数据源名 |
| data.executeReportDesc.dataSourceTag | String | 数据源标签 |
| data.executeReportDesc.envId | Integer | 执行环境 |
| data.executeReportDesc.envName | String | 执行环境名 |
| data.executeReportDesc.executeRecordName | String | 执行记录名称 |
| data.executeReportDesc.createTime | Long | 创建时间 |
| data.executeReportDesc.endTime | Long | 结束时间 |
| data.executeReportDesc.taskHasSuiteType | JSONArray | 执行记录类型（Integer 列表） |
| data.executeReportDesc.taskStatus | Integer | 任务状态 |
| data.executeReportDataView | JSONObject | 报告统计字段（ExecuteReportDataView） |
| data.executeReportDataView.caseTotal | Integer | 用例总数 |
| data.executeReportDataView.completeCaseTotal | Integer | 已完成用例总数 |
| data.executeReportDataView.successCaseTotal | Integer | 成功用例总数 |
| data.executeReportDataView.failCaseTotal | Integer | 失败用例总数 |
| data.executeReportDataView.effectiveExecuteTime | Long | 有效执行时长 |
| data.executeReportDataView.errorReportTotal | Integer | 误报数量 |
| data.executeReportDataView.defectCaseTotal | Integer | 缺陷数量 |
| data.executeReportOverview | JSONObject | 报告图表信息（ExecuteReportOverview） |
| data.executeReportOverview.executeCaseChart | JSONObject | 用例图表（ExecuteReportCaseChart） |
| data.executeReportOverview.executeCaseChart.title | String | 图表标题 |
| data.executeReportOverview.executeCaseChart.total | JSONObject | 汇总项（ExecuteRecordReportChartDetailDTO） |
| data.executeReportOverview.executeCaseChart.total.key | String | 分类key |
| data.executeReportOverview.executeCaseChart.total.value | Long | 数量 |
| data.executeReportOverview.executeCaseChart.list | JSONArray | 明细项列表（ExecuteRecordReportChartDetailDTO） |
| data.executeReportOverview.executeCaseChart.list[].key | String | 分类key |
| data.executeReportOverview.executeCaseChart.list[].value | Long | 数量 |
| data.executeReportOverview.executeCaseTestResultChart | JSONObject | 用例测试结果图表（ExecuteReportCaseTestResultChart） |
| data.executeReportOverview.executeCaseTestResultChart.title | String | 图表标题 |
| data.executeReportOverview.executeCaseTestResultChart.total | JSONObject | 汇总项（ExecuteRecordReportChartDetailDTO） |
| data.executeReportOverview.executeCaseTestResultChart.total.key | String | 分类key |
| data.executeReportOverview.executeCaseTestResultChart.total.value | Long | 数量 |
| data.executeReportOverview.executeCaseTestResultChart.list | JSONArray | 明细项列表（ExecuteRecordReportChartDetailDTO） |
| data.executeReportOverview.executeCaseTestResultChart.list[].key | String | 分类key |
| data.executeReportOverview.executeCaseTestResultChart.list[].value | Long | 数量 |

### 实现意图

聚合展示一份执行记录的宏观报告视图，供前端报告页面使用。

---

## 2. GET /v3/real_task/case_report/fail_case_detail — 失败用例图表

### 入口

`TaskExecuteRecordCaseReportController.failCaseDetail(@RequestParam task_execute_record_id)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_execute_record_id | Integer | 是 | 执行记录id（@RequestParam） |

### 响应结构

`ResponseResult<FailCaseDetailResponse>`，含：
- 失败用例按错误类型分布
- 失败用例按设备分布
- 失败用例按脚本分布
- 重试次数分布

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.failCaseDetails | JSONArray | 错误图表列（FailCaseDetail） |
| data.failCaseDetails.errorCode | Integer | 失败结果类型 |
| data.failCaseDetails.errorMsg | String | 失败结果 msg |
| data.failCaseDetails.errorMessage | String | 错误信息 |
| data.failCaseDetails.count | Integer | 失败数量 |
| data.failCaseDetails.source | String | 来源 |
| data.failTotal | Integer | 总数 |
| data.userErrorTotal | Integer | 自定义错误数量 |
| data.sysErrorTotal | Integer | 系统错误数量 |

### 实现意图

为前端图表组件提供失败用例的多维度分布数据，辅助问题定位。

## 涉及表汇总

| 表 | 说明 |
|----|------|
| `task_execute_record` | 执行记录主表 |
| `task_execute_record_report` | 执行报告 |
| `task_execute_record_report_case` | 报告用例结果 |
| `task_execute_record_case_step` | 用例步骤记录 |
| `task_execute_record_script` | 脚本执行记录 |
| `task_execute_record_device` | 设备执行记录 |
