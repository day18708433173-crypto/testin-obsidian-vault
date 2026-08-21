# TaskExecuteRecordCaseController — 用例执行记录管理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/task/TaskExecuteRecordCaseController.java`
> 类级路由：`/real_task`
> Service 实现：`cn.testin.service.impl.task.TaskExecuteRecordServiceImpl`（部分方法）、`cn.testin.service.impl.task.TaskExecuteRecordReportCaseServiceImpl`（1781 行）
> 业务：用例级执行记录的查询与管理——记录查询、报告用例列表、错误类型/缺陷平台标记、步骤详情、设备信息、失败用例分布统计、输入参数刷新、错误截图获取。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 1 | POST | `/v3/real_task/case/task_execute_record` | listTaskExecuteRecord | 查询用例任务执行记录 |
| 2 | POST | `/v3/real_task/case/list_report_case` | listTaskExecuteRecordCaseByCondition | 查询用例执行报告列表 |
| 3 | POST | `/v3/real_task/case/list_report_case_simple` | listSimpleTaskExecuteRecordCaseByCondition | 查询用例执行报告列表（简版） |
| 4 | DELETE | `/v3/real_task/case/{task_execute_record_id}` | deleteCaseRecord | 删除用例执行记录 |
| 5 | PUT | `/v3/real_task/case/modify_report_case/{task_execute_record_report_case_id}` | modifyRecordCase | 修改单条用例的错误类型/原因 |
| 6 | POST | `/v3/real_task/case/modify_report_case_batch` | modifyRecordCase | 批量修改用例错误类型/原因 |
| 7 | PUT | `/v3/real_task/case/modify_report_case_defect_platform/{task_execute_record_report_case_id}` | modifyRecordCaseDefectPlatformId | 修改缺陷平台关联ID |
| 8 | GET | `/v3/real_task/case/report_case_detail/{task_execute_record_report_case_id}` | reportCaseDetail | 查用例步骤报告详情 |
| 9 | GET | `/v3/real_task/case/step_device` | executeCaseDevice | 查用例步骤设备信息（GET） |
| 10 | POST | `/v3/real_task/case/step_device` | executeCaseDevice | 查用例步骤设备信息（POST） |
| 11 | GET | `/v3/real_task/case/{task_execute_record_id}` | getCaseRecord | 获取用例任务执行记录 |
| 12 | POST | `/v3/real_task/case/report_case_info` | reportCaseInfo | 内部接口：查询执行用例信息 |
| 13 | GET | `/v3/real_task/case/execute_case_statistic` | executeCaseStatistic | 查询用例执行详情统计 |
| 14 | POST | `/v3/real_task/case/execute_result/flush` | flushExecuteResult | 刷新执行结果 |
| 15 | GET | `/v3/real_task/case/fail_case` | failCaseDistribution | 失败用例分布统计 |
| 16 | GET | `/v3/real_task/case/fail_case_detail` | failCaseDetailDistribution | 失败用例详细信息 |
| 17 | POST | `/v3/real_task/case/refresh_report_input_param` | updateReportInputParam | 刷新报告输入参数 |
| 18 | POST | `/v3/real_task/case/get_report_error_images` | getReportErrorImages | 获取报告错误截图 |

---

## 1. POST /v3/real_task/case/task_execute_record — 查询用例任务执行记录

### 入口

`TaskExecuteRecordCaseController.listTaskExecuteRecord(@RequestBody TaskExecuteRecordRequest request)`

### 请求参数（TaskExecuteRecordRequest，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 否 | 项目ID |
| userId | Integer | 否 | 用户ID |
| userName | String | 否 | 用户名 |
| eid | Integer | 否 | 企业ID |
| executeRecordName | String | 否 | 执行记录名称 |
| taskExecuteRecordType | JSONArray | 否 | 待测试应用类型（Integer，多选） |
| taskStatus | Integer | 否 | 执行状态 |
| createUserName | String | 否 | 创建人 |
| taskTemplateId | Integer | 否 | 任务模板ID |
| id | Integer | 否 | 执行记录ID |
| startCreateTime | String | 否 | 创建开始时间（LocalDate） |
| endCreateTime | String | 否 | 创建结束时间（LocalDate） |
| startCreateTimeStamp | Long | 否 | 创建开始时间戳 |
| endCreateTimeStamp | Long | 否 | 创建结束时间戳 |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |

### 响应结构

`ResponseResult<BasePageListResponseDTO<TaskExecuteRecordResponse>>`

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |
| data.list | JSONArray | 执行记录列表（TaskExecuteRecordResponse） |
| data.list[].id | Integer | 主键 |
| data.list[].parentId | Integer | 父级ID |
| data.list[].taskName | String | 执行记录名称 |
| data.list[].caseTotal | Integer | 预期执行用例数 |
| data.list[].deviceTotal | Integer | 设备数量 |
| data.list[].effectiveExecuteTime | Long | 有效执行时间 |
| data.list[].taskStatus | Integer | 执行状态 |
| data.list[].successCaseTotal | Integer | 通过用例数 |
| data.list[].failCaseTotal | Integer | 失败用例数（含超时） |
| data.list[].waitCaseTotal | Integer | 待执行用例数 |
| data.list[].runningCaseTotal | Integer | 执行中的用例数 |
| data.list[].completeCaseTotal | Integer | 已执行用例数 |
| data.list[].timeoutCaseTotal | Integer | 超时用例数 |
| data.list[].skipCaseTotal | Integer | 跳过用例数 |
| data.list[].cancelCaseTotal | Integer | 取消用例数 |
| data.list[].createUserId | Integer | 创建人ID |
| data.list[].createUserName | String | 创建人名称 |
| data.list[].createTime | Long | 创建时间 |
| data.list[].updateTime | Long | 更新时间 |
| data.list[].endTime | Long | 结束时间 |
| data.list[].projectId | Integer | 项目ID |
| data.list[].next | JSONArray | 子节点列表（TaskExecuteRecordResponse） |
| data.list[].taskExecuteRecordType | JSONArray | 执行记录类型（Integer） |
| data.list[].dataSourceId | Integer | 数据源ID |

关联表：`task_execute_record`、`task_execute_record_case`

---

## 2. POST /v3/real_task/case/list_report_case — 查询用例执行报告列表

### 入口

`TaskExecuteRecordCaseController.listTaskExecuteRecordCaseByCondition(@RequestBody TaskExecuteRecordReportCaseRequest request)`

### 请求参数（TaskExecuteRecordReportCaseRequest，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 否 | 项目ID |
| userId | Integer | 否 | 用户ID |
| userName | String | 否 | 用户名 |
| eid | Integer | 否 | 企业ID |
| reportCaseIds | JSONArray | 否 | 报告用例ID列表（Long） |
| taskExecuteRecordId | Integer | 否 | 执行记录ID |
| taskExecuteRecordIds | JSONArray | 否 | 执行记录ID集合（Integer） |
| caseId | Integer | 否 | 用例ID |
| caseName | String | 否 | 用例名称 |
| caseTagList | JSONArray | 否 | 用例标签（String） |
| executeStatus | Integer | 否 | 执行状态 |
| testResult | Integer | 否 | 测试结果 |
| errorCode | Integer | 否 | 问题类型 |
| errorMessage | String | 否 | 错误原因 |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |

### 响应结构

`ResponseResult<BasePageListResponseDTO<TaskExecuteRecordReportCaseResponse>>`

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |
| data.list | JSONArray | 用例报告列表（TaskExecuteRecordReportCaseResponse） |
| data.list[].id | Long | 报告用例ID |
| data.list[].taskName | String | 所在任务名称 |
| data.list[].caseId | Integer | 用例ID |
| data.list[].caseName | String | 用例名称 |
| data.list[].caseTagList | JSONArray | 用例标签（String） |
| data.list[].execStatus | Integer | 执行状态 |
| data.list[].testResult | Integer | 测试结果 |
| data.list[].errorCode | Integer | 问题类型 |
| data.list[].executeCostTime | Long | 用例执行耗时 |
| data.list[].errorCodeName | String | 错误原因 |
| data.list[].createTime | Long | 创建时间 |
| data.list[].executeEndTime | Long | 结束时间 |
| data.list[].dataParams | String | 概要数据 |
| data.list[].errorMessage | String | 错误信息 |
| data.list[].taskTemplateId | Integer | 模板ID |
| data.list[].defectPlatformId | Integer | 绑定的缺陷ID |
| data.list[].taskExecuteRecordId | Integer | 执行记录ID |
| data.list[].errorStepImageUrl | String | 失败步骤截图地址 |
| data.list[].errorAfterStepImageUrl | String | 失败步骤后截图地址 |

关联表：`task_execute_record_report_case`、`task_execute_record_report`

---

## 3. POST /v3/real_task/case/list_report_case_simple — 查询用例执行报告列表（简版）

### 入口

`TaskExecuteRecordCaseController.listSimpleTaskExecuteRecordCaseByCondition(@RequestBody TaskExecuteRecordReportCaseRequest request)`

### 请求参数（TaskExecuteRecordReportCaseRequest，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 否 | 项目ID |
| userId | Integer | 否 | 用户ID |
| userName | String | 否 | 用户名 |
| eid | Integer | 否 | 企业ID |
| reportCaseIds | JSONArray | 否 | 报告用例ID列表（Long） |
| taskExecuteRecordId | Integer | 否 | 执行记录ID |
| taskExecuteRecordIds | JSONArray | 否 | 执行记录ID集合（Integer） |
| caseId | Integer | 否 | 用例ID |
| caseName | String | 否 | 用例名称 |
| caseTagList | JSONArray | 否 | 用例标签（String） |
| executeStatus | Integer | 否 | 执行状态 |
| testResult | Integer | 否 | 测试结果 |
| errorCode | Integer | 否 | 问题类型 |
| errorMessage | String | 否 | 错误原因 |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |

### 响应结构

`ResponseResult<BasePageListResponseDTO<TaskExecuteRecordReportCaseResponse>>`

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |
| data.list | JSONArray | 用例报告列表（TaskExecuteRecordReportCaseResponse） |

> 元素字段同「查询用例执行报告列表」。

关联表：`task_execute_record_report_case`、`task_execute_record_report`

---

## 4. DELETE /v3/real_task/case/{task_execute_record_id} — 删除用例执行记录

### 入口

`TaskExecuteRecordCaseController.deleteCaseRecord(@PathVariable taskExecuteRecordId)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_execute_record_id | Integer | 是 | 执行记录ID（路径变量） |

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 影响行数。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 影响行数 |

---

## 5. PUT /v3/real_task/case/modify_report_case/{task_execute_record_report_case_id} — 修改单条用例错误类型/原因

### 入口

`TaskExecuteRecordCaseController.modifyRecordCase(@PathVariable taskExecuteRecordReportCaseId, @RequestBody ModifyTaskCaseRequest request)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_execute_record_report_case_id | Integer | 是 | 报告用例ID（路径变量） |
| projectId | Integer | 否 | 项目ID |
| userId | Integer | 否 | 用户ID |
| userName | String | 否 | 用户名 |
| eid | Integer | 否 | 企业ID |
| id | Integer | 否 | report_case_id（路径变量回填） |
| ids | JSONArray | 否 | report_case_id 列表（Long） |
| errorCode | Integer | 否 | 自定义错误码 |
| errorMessage | String | 否 | 自定义错误原因 |
| testResult | Integer | 否 | 测试结果 |
| defectPlatformId | Integer | 否 | 缺陷平台ID |
| taskExecuteRecordId | Integer | 否 | 执行记录ID |
| executeRecordTaskId | Long | 否 | 测试计划的执行记录ID |
| condition | JSONObject | 否 | 查询条件（TaskExecuteRecordReportCaseRequest） |
| condition.reportCaseIds | JSONArray | 否 | 报告用例ID列表（Long） |
| condition.taskExecuteRecordId | Integer | 否 | 执行记录id |
| condition.taskExecuteRecordIds | JSONArray | 否 | 执行记录id集合（Integer） |
| condition.caseId | Integer | 否 | 用例id |
| condition.caseName | String | 否 | 用例名称 |
| condition.caseTagList | JSONArray | 否 | 用例标签（String） |
| condition.executeStatus | Integer | 否 | 执行状态 |
| condition.testResult | Integer | 否 | 测试结果 |
| condition.errorCode | Integer | 否 | 问题类型 |
| condition.errorMessage | String | 否 | 错误原因 |
| condition.page | Integer | 否 | 当前页 |
| condition.pageSize | Integer | 否 | 页大小 |
| condition.projectId | Integer | 否 | 项目ID（继承 BaseRequestDTO） |
| condition.userId | Integer | 否 | 用户ID（继承 BaseRequestDTO） |
| condition.eid | Integer | 否 | 企业ID（继承 BaseRequestDTO） |

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 影响行数。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 影响行数 |

关联表：`task_execute_record_report_case`

---

## 6. POST /v3/real_task/case/modify_report_case_batch — 批量修改用例错误类型/原因

### 入口

`TaskExecuteRecordCaseController.modifyRecordCase(@RequestBody ModifyTaskCaseRequest request)`

### 请求参数（ModifyTaskCaseRequest，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 否 | 项目ID |
| userId | Integer | 否 | 用户ID |
| userName | String | 否 | 用户名 |
| eid | Integer | 否 | 企业ID |
| id | Integer | 否 | report_case_id |
| ids | JSONArray | 否 | report_case_id 列表（Long） |
| errorCode | Integer | 否 | 自定义错误码 |
| errorMessage | String | 否 | 自定义错误原因 |
| testResult | Integer | 否 | 测试结果 |
| defectPlatformId | Integer | 否 | 缺陷平台ID |
| taskExecuteRecordId | Integer | 否 | 执行记录ID |
| executeRecordTaskId | Long | 否 | 测试计划的执行记录ID |
| condition | JSONObject | 否 | 查询条件（TaskExecuteRecordReportCaseRequest） |
| condition.reportCaseIds | JSONArray | 否 | 报告用例ID列表（Long） |
| condition.taskExecuteRecordId | Integer | 否 | 执行记录id |
| condition.taskExecuteRecordIds | JSONArray | 否 | 执行记录id集合（Integer） |
| condition.caseId | Integer | 否 | 用例id |
| condition.caseName | String | 否 | 用例名称 |
| condition.caseTagList | JSONArray | 否 | 用例标签（String） |
| condition.executeStatus | Integer | 否 | 执行状态 |
| condition.testResult | Integer | 否 | 测试结果 |
| condition.errorCode | Integer | 否 | 问题类型 |
| condition.errorMessage | String | 否 | 错误原因 |
| condition.page | Integer | 否 | 当前页 |
| condition.pageSize | Integer | 否 | 页大小 |
| condition.projectId | Integer | 否 | 项目ID（继承 BaseRequestDTO） |
| condition.userId | Integer | 否 | 用户ID（继承 BaseRequestDTO） |
| condition.eid | Integer | 否 | 企业ID（继承 BaseRequestDTO） |

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 影响行数。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 影响行数 |

关联表：`task_execute_record_report_case`

---

## 7. PUT /v3/real_task/case/modify_report_case_defect_platform/{task_execute_record_report_case_id} — 修改缺陷平台关联ID

### 入口

`TaskExecuteRecordCaseController.modifyRecordCaseDefectPlatformId(@PathVariable taskExecuteRecordReportCaseId, @RequestBody ModifyTaskCaseRequest request)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_execute_record_report_case_id | Integer | 是 | 报告用例ID（路径变量） |
| projectId | Integer | 否 | 项目ID |
| userId | Integer | 否 | 用户ID |
| userName | String | 否 | 用户名 |
| eid | Integer | 否 | 企业ID |
| defectPlatformId | Integer | 否 | 缺陷平台ID |
| ids | JSONArray | 否 | report_case_id 列表（Long） |
| errorCode | Integer | 否 | 自定义错误码 |
| errorMessage | String | 否 | 自定义错误原因 |
| testResult | Integer | 否 | 测试结果 |
| taskExecuteRecordId | Integer | 否 | 执行记录ID |

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 影响行数。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 影响行数 |

关联表：`task_execute_record_report_case`

---

## 8. GET /v3/real_task/case/report_case_detail/{task_execute_record_report_case_id} — 用例步骤报告详情

### 入口

`TaskExecuteRecordCaseController.reportCaseDetail(@PathVariable taskExecuteRecordReportCaseId)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_execute_record_report_case_id | Integer | 是 | 报告用例ID（路径变量） |

### 响应结构

`ResponseResult<ReportCaseDetailResponse>`，含每个步骤的输入/输出/截图/日志。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.stepList | JSONArray | 步骤列表（ExecuteCaseStepResponse） |
| data.stepList[].id | Long | 步骤ID |
| data.stepList[].caseId | Integer | 用例ID |
| data.stepList[].caseName | String | 用例名称 |
| data.stepList[].caseStepId | Integer | 用例步骤ID |
| data.stepList[].stepExpect | String | 步骤预期结果 |
| data.stepList[].scriptNo | Integer | 脚本编号 |
| data.stepList[].scriptType | Integer | 脚本类型 |
| data.stepList[].scriptName | String | 脚本名称 |
| data.stepList[].executeStatus | Integer | 执行状态 |
| data.stepList[].testResult | Integer | 测试结果 |
| data.stepList[].costTime | Long | 耗时 |
| data.stepList[].stepDesc | String | 步骤描述 |
| data.stepList[].deviceId | String | 设备ID |
| data.stepList[].deviceName | String | 设备名称 |
| data.stepList[].errorCode | Integer | 自定义类型 |
| data.stepList[].errorCodeName | String | 自定义问题描述 |
| data.stepList[].errorMessage | String | 错误原因 |
| data.stepList[].videoUrl | String | 视频链接 |
| data.stepList[].logUrl | String | 日志链接 |
| data.stepList[].executeEndTime | Long | 结束时间 |
| data.stepList[].createTime | Long | 创建时间 |
| data.stepList[].scriptStepCount | Integer | 脚本步骤数 |
| data.executeCostTime | Long | 花费时间 |
| data.taskExecuteRecordCaseId | Long | 用例执行记录ID |
| data.caseName | String | 用例名称 |
| data.dataSourceId | Integer | 数据表ID |
| data.dataSourceName | String | 数据源名称 |
| data.dataSourceTag | JSONArray | 数据源标签（String） |
| data.caseStepCount | Integer | 步骤数量 |
| data.testResult | Integer | 测试结果 |

关联表：`task_execute_record_report_case`、`task_execute_record_case_step`

---

## 9. GET /v3/real_task/case/step_device — 用例步骤设备信息（GET）

### 入口

`TaskExecuteRecordCaseController.executeCaseDevice(@RequestParam taskExecuteRecordReportId, @RequestParam taskExecuteRecordId)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_execute_record_report_id | Long | 是 | 执行记录报告ID（@RequestParam，默认必传） |
| task_execute_record_id | Integer | 是 | 执行记录ID（@RequestParam，默认必传） |

### 响应结构

`ResponseResult<List<DeviceDTO>>`

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | JSONArray | 设备列表（DeviceDTO） |
| data[].vhost | Integer | 设备上报到的平台 |
| data[].deviceId | String | 设备ID |
| data[].brandName | String | 品牌 |
| data[].modelName | String | 机型 |
| data[].releaseVersion | String | 系统版本 |
| data[].ucomId | String | 所在上位机ID |
| data[].ucomIp | String | 所在上位机IP |
| data[].networkType | Integer | 网络类型：0无网/1wifi/2mobile |
| data[].network | Integer | 0无网/1有网 |
| data[].debugMode | Integer | 是否支持真机调试：0不支持/1支持 |
| data[].status | Integer | 状态：0空闲/1运行中/2掉线/9未知 |
| data[].action | Integer | 动作：0空闲/1测试/2真机调试 |
| data[].modelAlias | String | 别名 |
| data[].location | String | 位置 |
| data[].dpiHeight | Integer | 屏幕高度 |
| data[].dpiWidth | Integer | 屏幕宽度 |
| data[].temperature | Double | 电池温度 |
| data[].descr | String | 设备备注 |
| data[].osName | Integer | 系统类型 |
| data[].webDeviceType | Integer | web 设备类型 |
| data[].webDeviceTypeName | String | web 设备资源类型名称 |
| data[].errorMsg | String | 错误原因 |
| data[].screenMode | Integer | 屏幕状态：0不息屏/1息屏 |
| data[].syspfName | String | 系统平台名称 |
| data[].source | String | 来源 |
| data[].deviceType | Integer | 设备类型 |

依赖：[任务调度服务](../../../任务调度服务（real-scheduling）/07-开放接口文档/00-模块索引.md)（设备信息查询）

---

## 10. POST /v3/real_task/case/step_device — 用例步骤设备信息（POST）

### 入口

`TaskExecuteRecordCaseController.executeCaseDevice(@RequestBody ReportCaseDeviceRequest request)`

### 请求参数（ReportCaseDeviceRequest，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| executeRecordIds | JSONArray | 否 | 执行记录ID列表（Integer） |
| reportCaseIds | JSONArray | 否 | 执行记录报告ID列表（Long） |

### 响应结构

`ResponseResult<List<CaseDeviceResponse>>`

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | JSONArray | 设备信息列表（CaseDeviceResponse） |
| data[].reportCaseId | Long | 用例执行报告ID |
| data[].deviceDTOList | JSONArray | 设备列表（DeviceDTO） |
| data[].deviceDTOList[].deviceId | String | 设备ID |
| data[].deviceDTOList[].brandName | String | 品牌 |
| data[].deviceDTOList[].modelName | String | 机型 |
| data[].deviceDTOList[].releaseVersion | String | 系统版本 |
| data[].deviceDTOList[].status | Integer | 状态 |
| data[].deviceDTOList[].deviceType | Integer | 设备类型 |

依赖：[任务调度服务](../../../任务调度服务（real-scheduling）/07-开放接口文档/00-模块索引.md)（设备信息查询）

---

## 11. GET /v3/real_task/case/{task_execute_record_id} — 获取用例任务执行记录

### 入口

`TaskExecuteRecordCaseController.getCaseRecord(@PathVariable taskExecuteRecordId)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_execute_record_id | Integer | 是 | 执行记录ID（路径变量） |

### 响应结构

`ResponseResult<TaskExecuteRecord>`（执行记录实体）

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据（TaskExecuteRecord 实体） |
| data.id | Integer | 执行记录ID |
| data.projectId | Integer | 项目ID |
| data.taskType | Integer | 任务类型 |
| data.suiteId | Integer | 应用ID |
| data.taskName | String | 任务名称 |
| data.taskStatus | Integer | 任务状态 |
| data.taskSource | Integer | 任务来源 |
| data.executeRecordTaskId | Long | 测试计划任务ID |
| data.executeRecordTaskName | String | 测试计划任务名称 |
| data.executeRecordId | Long | 关联计划执行记录ID |
| data.taskTemplateId | Integer | 关联模板ID |
| data.createUserId | Integer | 创建人ID |
| data.updateUserId | Integer | 更新人ID |
| data.createTime | Date | 创建时间 |
| data.updateTime | Date | 更新时间 |
| data.taskExecuteId | String | 任务执行ID（uuid） |
| data.parentId | Integer | 父节点ID |
| data.scriptTotal | Integer | 脚本总数 |
| data.executeScriptTotal | Integer | 执行脚本总数 |
| data.successScriptTotal | Integer | 成功脚本数 |
| data.failScriptTotal | Integer | 失败脚本数 |
| data.skipScriptTotal | Integer | 跳过脚本数 |
| data.cancelScriptTotal | Integer | 取消脚本数 |
| data.timeoutScriptTotal | Integer | 超时脚本数 |
| data.caseTotal | Integer | 用例总数 |
| data.executeCaseTotal | Integer | 执行用例总数 |
| data.successCaseTotal | Integer | 成功用例数 |
| data.failCaseTotal | Integer | 失败用例数 |
| data.skipCaseTotal | Integer | 跳过用例数 |
| data.cancelCaseTotal | Integer | 取消用例数 |
| data.timeoutCaseTotal | Integer | 超时用例数 |
| data.deviceTotal | Integer | 设备总数 |
| data.effectiveExecuteTime | Long | 有效执行时间 |
| data.errorMessage | String | 错误信息 |
| data.endTime | Date | 结束时间 |

---

## 12. POST /v3/real_task/case/report_case_info — 查询执行用例信息（内部接口）

### 入口

`TaskExecuteRecordCaseController.reportCaseInfo(@RequestBody ReportCaseInfoRequest request)`

### 请求参数（ReportCaseInfoRequest，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| caseIds | JSONArray | 否 | 用例ID列表（Integer） |

### 响应结构

`ResponseResult<BasePageListResponseDTO<ExecuteRecordReportCaseInfoResponse>>`

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |
| data.list | JSONArray | 用例信息列表（ExecuteRecordReportCaseInfoResponse） |
| data.list[].id | Long | 报告用例ID |
| data.list[].caseId | Integer | 用例ID |
| data.list[].caseName | String | 用例名称 |
| data.list[].executeRecordTaskId | Integer | 测试计划关联ID |

为其他服务（如 [平台基础功能服务](../../../平台基础功能服务/00-首页.md) 的测试计划）提供执行用例报告信息查询。

---

## 13. GET /v3/real_task/case/execute_case_statistic — 用例执行统计

### 入口

`TaskExecuteRecordCaseController.executeCaseStatistic(CaseStatisticDetailRequest request)` — `@UnderlineToCamel`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| case_name | String | 否 | 用例名称 |
| case_id | Integer | 否 | 用例ID |
| project_id | Integer | 否 | 项目ID |
| end_time_start | Long | 否 | 结束时间开始 |
| end_time_end | Long | 否 | 结束时间结束 |
| order_by_type | String | 否 | 排序类型 |
| order_by_col | String | 否 | 排序字段 |
| page | Integer | 否 | 当前页 |
| page_size | Integer | 否 | 每页大小 |

### 响应结构

`ResponseResult<BasePageListResponseDTO<CaseStatisticViewResponse>>`

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |
| data.list | JSONArray | 统计列表（CaseStatisticViewResponse） |
| data.list[].createCaseTotal | Integer | 创建用例总数 |
| data.list[].updateCaseTotal | Integer | 更新用例总数 |
| data.list[].executeCaseTotal | Integer | 执行用例总数 |
| data.list[].successCaseTotal | Integer | 成功用例总数 |
| data.list[].failCaseTotal | Integer | 失败用例总数 |
| data.list[].skipCaseTotal | Integer | 跳过用例总数 |
| data.list[].cancelCaseTotal | Integer | 取消用例总数 |
| data.list[].caseId | Integer | 用例ID |
| data.list[].caseName | String | 用例名称 |
| data.list[].passRate | Double | 通过率 |

按时间/项目/任务维度统计用例执行情况（通过/失败/跳过/超时分布）。

---

## 14. POST /v3/real_task/case/execute_result/flush — 刷新执行结果

### 入口

`TaskExecuteRecordCaseController.flushExecuteResult()`（无请求参数）

### 请求参数

无请求参数。

### 响应结构

`ResponseResult<Integer>`，`data` = 刷新的执行结果数量。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Integer | 刷新的执行结果数量 |

手动触发执行结果刷新（补偿机制：当自动结果回收出现异常时，手动刷新确保数据一致）。

---

## 15. GET /v3/real_task/case/fail_case — 失败用例分布统计

### 入口

`TaskExecuteRecordCaseController.failCaseDistribution(CaseStatisticRequestDTO requestDTO)` — `@UnderlineToCamel`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| project_id | Integer | 否 | 项目ID |
| start_time | Long | 否 | 开始时间 |
| end_time | Long | 否 | 结束时间 |
| error_code | Integer | 否 | 错误码 |
| error_message | String | 否 | 错误信息 |
| page | Integer | 否 | 当前页 |
| page_size | Integer | 否 | 每页大小 |

### 响应结构

`ResponseResult<FailCaseDetailResponse>`

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.failCaseDetails | JSONArray | 错误图表列（FailCaseDetail） |
| data.failCaseDetails[].errorCode | Integer | 失败结果类型 |
| data.failCaseDetails[].errorMsg | String | 失败结果 msg |
| data.failCaseDetails[].errorMessage | String | 错误信息 |
| data.failCaseDetails[].count | Integer | 失败数量 |
| data.failCaseDetails[].source | String | 来源 |
| data.failTotal | Integer | 总数 |
| data.userErrorTotal | Integer | 自定义错误数量 |
| data.sysErrorTotal | Integer | 系统错误数量 |

返回失败用例按维度的分布统计（如按错误类型、按设备、按脚本）。

---

## 16. GET /v3/real_task/case/fail_case_detail — 失败用例详细信息

### 入口

`TaskExecuteRecordCaseController.failCaseDetailDistribution(CaseStatisticRequestDTO requestDTO)` — `@UnderlineToCamel`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| project_id | Integer | 否 | 项目ID |
| start_time | Long | 否 | 开始时间 |
| end_time | Long | 否 | 结束时间 |
| error_code | Integer | 否 | 错误码 |
| error_message | String | 否 | 错误信息 |
| page | Integer | 否 | 当前页 |
| page_size | Integer | 否 | 每页大小 |

### 响应结构

`ResponseResult<BasePageListResponseDTO<FailCaseDetail>>`

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |
| data.list | JSONArray | 失败用例列表（FailCaseDetail） |
| data.list[].errorCode | Integer | 失败结果类型 |
| data.list[].errorMsg | String | 失败结果 msg |
| data.list[].errorMessage | String | 错误信息 |
| data.list[].count | Integer | 失败数量 |
| data.list[].source | String | 来源 |

返回具体失败用例列表。

---

## 17. POST /v3/real_task/case/refresh_report_input_param — 刷新报告输入参数

### 入口

`TaskExecuteRecordCaseController.updateReportInputParam(@RequestBody TaskExecuteRecordReportCaseRequest request)`

### 请求参数（TaskExecuteRecordReportCaseRequest，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 否 | 项目ID |
| userId | Integer | 否 | 用户ID |
| userName | String | 否 | 用户名 |
| eid | Integer | 否 | 企业ID |
| reportCaseIds | JSONArray | 否 | 报告用例ID列表（Long） |
| taskExecuteRecordId | Integer | 否 | 执行记录ID |
| taskExecuteRecordIds | JSONArray | 否 | 执行记录ID集合（Integer） |
| caseId | Integer | 否 | 用例ID |
| caseName | String | 否 | 用例名称 |
| caseTagList | JSONArray | 否 | 用例标签（String） |
| executeStatus | Integer | 否 | 执行状态 |
| testResult | Integer | 否 | 测试结果 |
| errorCode | Integer | 否 | 问题类型 |
| errorMessage | String | 否 | 错误原因 |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 刷新数量。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 刷新数量 |

更新报告用例中记录的输入参数（用例执行时的动态参数）。

---

## 18. POST /v3/real_task/case/get_report_error_images — 获取报告错误截图

### 入口

`TaskExecuteRecordCaseController.getReportErrorImages(@RequestBody TaskExecuteRecordReportCaseRequest request)`

### 请求参数（TaskExecuteRecordReportCaseRequest，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 否 | 项目ID |
| userId | Integer | 否 | 用户ID |
| userName | String | 否 | 用户名 |
| eid | Integer | 否 | 企业ID |
| reportCaseIds | JSONArray | 否 | 报告用例ID列表（Long） |
| taskExecuteRecordId | Integer | 否 | 执行记录ID |
| taskExecuteRecordIds | JSONArray | 否 | 执行记录ID集合（Integer） |
| caseId | Integer | 否 | 用例ID |
| caseName | String | 否 | 用例名称 |
| caseTagList | JSONArray | 否 | 用例标签（String） |
| executeStatus | Integer | 否 | 执行状态 |
| testResult | Integer | 否 | 测试结果 |
| errorCode | Integer | 否 | 问题类型 |
| errorMessage | String | 否 | 错误原因 |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |

### 响应结构

`ResponseResult<TaskExecuteRecordReportCaseResponse>`，含失败截图地址。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据（TaskExecuteRecordReportCaseResponse） |
| data.id | Long | 报告用例ID |
| data.taskName | String | 所在任务名称 |
| data.caseId | Integer | 用例ID |
| data.caseName | String | 用例名称 |
| data.caseTagList | JSONArray | 用例标签（String） |
| data.execStatus | Integer | 执行状态 |
| data.testResult | Integer | 测试结果 |
| data.errorCode | Integer | 问题类型 |
| data.executeCostTime | Long | 用例执行耗时 |
| data.errorCodeName | String | 错误原因 |
| data.createTime | Long | 创建时间 |
| data.executeEndTime | Long | 结束时间 |
| data.dataParams | String | 概要数据 |
| data.errorMessage | String | 错误信息 |
| data.taskTemplateId | Integer | 模板ID |
| data.defectPlatformId | Integer | 绑定的缺陷ID |
| data.taskExecuteRecordId | Integer | 执行记录ID |
| data.errorStepImageUrl | String | 失败步骤截图地址 |
| data.errorAfterStepImageUrl | String | 失败步骤后截图地址 |

获取用例执行失败时的错误截图/日志截图。

## 涉及表汇总

| 表 | 说明 |
|----|------|
| `task_execute_record` | 执行记录主表 |
| `task_execute_record_case` | 执行记录-用例关联 |
| `task_execute_record_report_case` | 报告-用例结果 |
| `task_execute_record_case_step` | 用例步骤执行记录 |
| `task_execute_record_case_tag` | 用例标签 |
| `task_execute_record_report` | 执行报告主表 |
