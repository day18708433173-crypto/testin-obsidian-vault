# TaskExecuteRecordStandardDetailController — 执行标准详情

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/task/TaskExecuteRecordStandardDetailController.java`
> 类级路由：`/real_task`
> Service 实现：`cn.testin.service.impl.task.TaskExecuteRecordStandardDetailServiceImpl`
> 业务：任务执行标准详情（execStandard 配置）的分页查询。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 1 | GET | `/v3/real_task/task_execute_record_standard_detail/list` | listRecordStandardDetail | 分页查询执行标准详情列表 |

---

## 1. GET /v3/real_task/task_execute_record_standard_detail/list — 执行标准详情列表

### 入口

`TaskExecuteRecordStandardDetailController.listRecordStandardDetail(TaskExecuteRecordStandardDetailRequestDTO requestDTO)` — `@UnderlineToCamel`

### 请求参数（TaskExecuteRecordStandardDetailRequestDTO，Query）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskExecuteRecordId | Integer | 否 | 执行记录id（可选筛选） |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 每页大小 |

### 响应结构

`ResponseResult<BasePageListResponseDTO<TaskExecuteRecordStandardDetailResponse>>`

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
| data.list | JSONArray | 当前页数据列表 |
| data.list.id | Integer | 主键 |
| data.list.taskExecuteRecordId | Integer | 关联的执行记录id |
| data.list.coverInstall | Integer | 执行前卸载安装 |
| data.list.overwriteInstall | Integer | 执行前覆盖安装 |
| data.list.cleanData | Integer | 执行后清理数据 |
| data.list.uninstall | Integer | 执行后不卸载 app |
| data.list.install | Integer | 安装应用 |
| data.list.startUp | Integer | 启动应用 |
| data.list.keepApp | Integer | 执行后关闭应用 |
| data.list.video | Integer | 是否录制视频 |
| data.list.resign | Integer | iOS 重签配置 |
| data.list.taskExecuteMode | Integer | 上位机执行形式 |
| data.list.terminationOnError | Integer | 错误是否终止后续脚本 |
| data.list.stepGlobalTimeout | Integer | 步骤全局超时（毫秒） |
| data.list.customFilePath | String | 日志采集位置 |
| data.list.logCollection | Integer | 是否记录日志 |
| data.list.performanceDataCollection | Integer | 是否记录性能数据 |
| data.list.traversalTime | Long | 遍历时长 |
| data.list.monkeyTime | Long | monkey 时长 |
| data.list.retryNum | Integer | 脚本失败重测次数 |
| data.list.appStepGlobalTimeOut | Integer | app 步骤全局超时 |
| data.list.webStepGlobalTimeOut | Integer | web 步骤全局超时 |
| data.list.pcStepGlobalTimeOut | Integer | pc 步骤全局超时 |
| data.list.androidGlobalControlAccelerated | Integer | 全局智能加速 |
| data.list.failStepTexts | Integer | 是否返回失败截图文本 |

### 实现意图

执行标准（execStandard）定义了测试任务的执行参数阈值（如超时时间、重试次数、断言规则等）。本接口分页查询这些标准的配置详情。

### 关联表

`task_execute_record_standard_detail`

## 涉及表汇总

| 表 | 说明 |
|----|------|
| `task_execute_record_standard_detail` | 执行标准详情 |
| `task_template_standard_detail` | 模板级执行标准详情 |
