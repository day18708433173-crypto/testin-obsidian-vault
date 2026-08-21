# ProblemAnalysisReportController — 问题分析报告

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/mvc/controller/ProblemAnalysisReportController.java`
> 类级路由：`/realweb`
> Service 实现：`mvc/service/ReportService`

## 接口列表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|---|---|---|---|
| 1 | POST | `/v3/realweb/report/script_list` | getNeedAnalysisScriptList | 问题分析脚本列表（分页） |
| 2 | GET | `/v3/realweb/report/error_step_report_detail_list` | getErrorScriptReportDetailList | 错误步骤报告详情 |
| 3 | GET | `/v3/realweb/report/device_list/{task_id}` | getDeviceList | 任务设备列表 |
| 4 | POST | `/v3/realweb/report/refresh_report_input_param` | updateReportInputParam | 刷新报告输入参数 |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`。

---

## 1. POST /v3/realweb/report/script_list — 问题分析脚本列表

### 入口

`ProblemAnalysisReportController.getNeedAnalysisScriptList(@RequestBody NeedAnalysisScriptListRequestDTO requestDTO)`

### 请求参数（NeedAnalysisScriptListRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskId | String | 是 | 任务ID（空抛 GeneralException "无效的task_id"） |
| projectId | Integer | 否 | 项目ID |
| page / pageSize | Integer | 否 | 分页（默认1/50） |
| scriptNo | String | 否 | 脚本编号 |
| scriptName | String | 否 | 脚本名称 |
| testResult | String | 否 | 测试结果过滤 |
| errorCauseTypeId | Integer | 否 | 错误原因类型过滤 |
| customizeErrorMsg | String | 否 | 自定义错误消息 |
| deviceId | String | 否 | 设备ID |
| systemError | String | 否 | 系统错误 |
| isAnalyze | Boolean | 否 | 是否已分析（errorCauseTypeId存在/不存在） |
| resultCategory | String | 否 | 结果分类 |
| deviceType | String | 否 | 设备类型 |
| deviceVersion | String | 否 | 设备版本 |
| deviceIp | String | 否 | 设备IP |

### 响应结构

`ResponseResult<PageResponseDTO<ScriptProblemAnalysisDTO>>`。

ScriptProblemAnalysisDTO 字段：`taskId, subTaskId, subSubTaskId, scriptNo, scriptName, inputParams, reportDevice（含 webDeviceType/osName）, customizeErrorCauseMessage, errorCauseTypeId, resultCategory, errorCauseMessage, testResult`。

### 实现意图

分页查询 `PmScriptRunInfo` 中按条件筛选的脚本记录，排序规则：
- Web任务（taskId 以 `wt` 开头）：按 `subtaskid` + `webScript.orderNum` 排序
- PC任务（taskId 以 `pt` 开头）：按 `subtaskid` + `pcScript.orderNum` 排序
- DATA执行标准：按 `scriptNo/rowId` 排序

查询后关联 `PmReportDetail` 获取报告设备信息和错误详情。

### 调用链

```
ProblemAnalysisReportController.getNeedAnalysisScriptList
└─ ReportService.getNeedAnalysisScriptList(requestDTO)
   ├─ IPmScriptRunInfoDAO.baseList(condition) → MongoDB PmScriptRunInfo
   ├─ IPmTaskDetailDAO.get(taskId) → 取 execStandard 判断排序规则
   └─ IReportService.baseList(subSubTaskIds) → IPmReportDetailDAO.baseList
      → 组装 ScriptProblemAnalysisDTO
```

### 涉及表

| 存储 | 集合 | 操作 |
|------|------|------|
| MongoDB | PmScriptRunInfo | 读 |
| MongoDB | PmTaskDetail | 读 |
| MongoDB | PmReportDetail | 读 |

---

## 2. GET /v3/realweb/report/error_step_report_detail_list — 错误步骤报告详情

### 入口

`ProblemAnalysisReportController.getErrorScriptReportDetailList(@RequestParam("task_id") String taskId, @RequestParam("subsubtask_id") String subSubTaskId, @RequestParam(value="size", defaultValue="5") Integer size)`

### 请求参数

| 字段 | 来源 | 必填 | 说明 |
|------|------|------|------|
| task_id | Query | 是 | 任务ID |
| subsubtask_id | Query | 是 | 子子任务ID |
| size | Query | 否 | 返回步骤数（默认5） |

### 响应结构

`ResponseResult<Map<String, Object>>`，map 包含 `list`（步骤列表）和 `totalRow`（总数）。

### 实现意图

根据 scriptRunInfo 中 `warningTags` 的 `stepPath` 范围，对范围内每个步骤查询 `IReportService.stepdetail`（需要查询 `PmReportDetail.details` + `PmScriptRunInfo` + `PmScriptSummary` 重构失败步骤详情）。

### 调用链

```
ProblemAnalysisReportController.getErrorScriptReportDetailList
└─ ReportService.getErrorStepReportDetailList(taskId, subSubTaskId, size)
   ├─ IPmReportDetailDAO.get(taskId, subSubTaskId) → MongoDB
   ├─ 从 PmScriptRunInfo.warningTags 获取 stepPath 范围
   └─ IReportService.stepdetail(...) × N → PmReportDetail.details + PmScriptRunInfo + PmScriptSummary
```

---

## 3. GET /v3/realweb/report/device_list/{task_id} — 任务设备列表

### 入口

`ProblemAnalysisReportController.getDeviceList(@PathVariable("task_id") String taskId)`

### 请求参数

| 字段 | 来源 | 必填 | 说明 |
|------|------|------|------|
| task_id | Path | 是 | 任务ID |

### 响应结构

`ResponseResult<List<Map<String,String>>>`。

### 实现意图

从 `PmScriptRunInfo` 中聚合去重获取任务使用的设备列表（device type/version）。

### 调用链

```
ProblemAnalysisReportController.getDeviceList
└─ ReportService.getDeviceList(taskId)
   └─ IPmScriptRunInfoDAO.getDevicesByTaskId(taskId) → MongoDB 聚合
```

---

## 4. POST /v3/realweb/report/refresh_report_input_param — 刷新报告输入参数

### 入口

`ProblemAnalysisReportController.updateReportInputParam(@RequestBody UpdateReportInputParamDTO requestDTO)`

### 请求参数（UpdateReportInputParamDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskId | String | 否 | 主任务ID（与 taskIds 至少一个，否则返回 0） |
| subTaskId | String | 否 | 子任务ID（本接口未使用） |
| subSubTaskId | String | 否 | 子子任务ID（本接口未使用） |
| taskIds | List&lt;String&gt; | 否 | 额外任务ID列表（与 taskId 至少一个） |

### 响应结构

`ResponseResult<BaseResponseDTO>`，data.result = 成功更新的记录数。

### 实现意图

批量刷新报告中的输入参数信息：遍历 taskId + taskIds 合集，分页（每100条）读取 `PmReportDetail`，将 `inputParams` 重置为 `originalInputParams`（如果为空则保存原值），调用 `IReportService.handleInPutParam` 从 [ParameterSourceApi](../其他ApiServlet/service-ParameterSourceApi.md)（→DataSource服务）获取最新数据源配置（sourceConfigId/Name/ParentId），更新报告记录。

### 调用链

```
ProblemAnalysisReportController.updateReportInputParam
└─ ReportService.updateReportInputParam(requestDTO)
   ├─ IPmReportDetailDAO.baseList(每100条分页) → MongoDB
   └─ IReportService.handleInPutParam(taskId, reports)
      ├─ ParameterSourceApi.listScriptParamSourceNew → DataSource服务
      └─ ParameterSourceApi.getParamTableInfo → DataSource服务 POST /v3/datasource/executive_summaries
```

---

## 备注

- `getNeedAnalysisScriptList` 的排序逻辑区分 Web/PC 端和 DATA 执行标准。
- `getErrorScriptReportDetailList` 利用 `warningTags.stepPath` 定位错误步骤范围。
- `updateReportInputParam` 用于数据源配置变更后同步刷新已有报告的参数展示。

相关文档：[00-分支索引](00-分支索引.md) · [ReportController](ReportController.md)
