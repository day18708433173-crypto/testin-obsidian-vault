# TaskReportSummaryController — 报告摘要（日志/性能/埋点）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/task/TaskReportSummaryController.java`
> 类级路由：`/real_task/summary`
> Service 实现：`cn.testin.service.impl.task.TaskReportSummaryServiceImpl`、`cn.testin.service.impl.task.TaskExecuteRecordReportServiceImpl`（部分方法）
> 业务：报告摘要视图——执行日志查询、性能数据、埋点信息。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 1 | GET | `/v3/real_task/summary/logs` | queryLogs | 查询执行日志 |
| 2 | GET | `/v3/real_task/summary/report_performance` | getPerformanceDetail | 查询性能详情 |
| 3 | POST | `/v3/real_task/summary/spot_info` | getSpotInformation | 查询埋点信息 |

---

## 1. GET /v3/real_task/summary/logs — 执行日志查询

### 入口

`TaskReportSummaryController.queryLogs(TaskReportLogQueryRequestDTO request)` — `@UnderlineToCamel`

### 请求参数（TaskReportLogQueryRequestDTO，Query）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| level | String | 否 | 日志级别 |
| keyword | String | 否 | 关键字 |
| pid | String | 否 | 进程id |
| startNum | Integer | 否 | 开始行 |
| pageSize | Integer | 否 | 每页行数 |
| taskExecuteRecordId | Integer | 否 | 执行记录id |
| taskExecuteRecordReportId | Long | 否 | 报告记录id |

### 响应结构

`ResponseResult<List<LogEntry>>`，含日志条目列表。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | JSONArray | 业务数据（日志条目列表） |
| data[].curNum | Integer | 当前行数 |
| data[].level | String | 日志级别 |
| data[].time | String | 时间 |
| data[].pid | String | 进程id |
| data[].tag | String | 标签 |
| data[].message | String | 日志内容 |

### 实现意图

从 [real-logfile](../../../平台基础功能服务/00-首页.md) 获取执行过程中的日志数据（脚本日志、设备日志、系统日志），按类型和步骤过滤。

### 调用链

```
TaskReportSummaryController.queryLogs
└─ TaskReportSummaryServiceImpl.queryLogs
   └─ FileDownloadApi → 外部服务/RealLogfile（日志文件获取）
```

---

## 2. GET /v3/real_task/summary/report_performance — 性能详情

### 入口

`TaskReportSummaryController.getPerformanceDetail(@Valid TaskReportPerformanceRequestDTO request)` — `@UnderlineToCamel`

### 请求参数（TaskReportPerformanceRequestDTO，Query，@Valid）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskExecuteRecordId | Integer | 否 | 执行记录id |
| taskExecuteRecordReportId | Long | 否 | 执行记录报告id |

### 响应结构

`ResponseResult<TaskReportPerformanceResponseDTO>`，含：
- CPU 使用率时间序列
- 内存使用时间序列
- 网络流量
- FPS 帧率数据
- 启动耗时等性能指标

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.cpuArray | JSONArray | CPU 数据（TaskReportPerformanceDetailDTO） |
| data.memArray | JSONArray | 内存数据（结构同 cpuArray） |
| data.ramTotal | Long | RAM 总量 |
| data.gpuArray | JSONArray | GPU 数据（结构同 cpuArray） |
| data.fpsArray | JSONArray | FPS 数据（结构同 cpuArray） |
| data.batteryArray | JSONArray | 电量数据（结构同 cpuArray） |
| data.netflowArray | JSONArray | 流量数据（结构同 cpuArray） |
| data.powerArray | JSONArray | 功耗数据（结构同 cpuArray） |
| data.cachedArray | JSONArray | 缓存数据（结构同 cpuArray） |
| data.cpuArray[].name | String | 曲线名称 |
| data.cpuArray[].genre | String | 曲线类型 |
| data.cpuArray[].data | JSONArray | 时间序列数据（ReportTimelineData） |
| data.cpuArray[].data[].timestamp | Long | 时间戳 |
| data.cpuArray[].data[].taskExecuteRecordReportId | Long | 任务执行记录报告id |
| data.cpuArray[].data[].value | Double | 具体的值 |
| data.cpuArray[].data[].line | Integer | 日志行号 |
| data.cpuArray[].data[].stepDesc | String | 步骤描述 |

### 实现意图

聚合展示移动端（App/Web/PC）执行过程的性能指标，对应性能测试场景。

---

## 3. POST /v3/real_task/summary/spot_info — 埋点信息查询

### 入口

`TaskReportSummaryController.getSpotInformation(@RequestBody TaskReportSpotRequestDTO request)`

### 请求参数（TaskReportSpotRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskExecuteRecordIds | JSONArray | 否 | 执行记录id列表 |

### 响应结构

`ResponseResult<List<SpotInformationResponse>>`

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | JSONArray | 业务数据（埋点信息列表） |
| data[].key | String | 唯一键 |
| data[].item | String | 埋点名称 |
| data[].actionTimeTotal | Double | 耗时 |
| data[].networkUpFlowTotal | Double | 上行流量 |
| data[].networkDownFlowTotal | Double | 下行流量 |
| data[].scriptName | String | 脚本名称 |
| data[].caseName | String | 用例名称 |
| data[].subSubTaskId | String | 子子任务id |
| data[].taskExecuteRecordReportId | Long | 用例执行记录id |
| data[].modelName | String | 执行设备 |
| data[].deviceVersion | String | 系统版本 |
| data[].appVersion | String | app 版本 |
| data[].netWork | String | 网络类型 |
| data[].startTime | Long | 开始时间 |
| data[].endTime | Long | 结束时间 |
| data[].link | String | 报告链接 |

### 实现意图

查询任务执行过程中记录的埋点事件（自定义打点），用于业务指标分析。

## 涉及表汇总

| 表 | 说明 |
|----|------|
| `task_execute_record` | 执行记录主表 |
| `task_execute_record_report` | 执行报告 |
| `task_execute_record_report_detail` | 报告详情（性能数据嵌入） |
