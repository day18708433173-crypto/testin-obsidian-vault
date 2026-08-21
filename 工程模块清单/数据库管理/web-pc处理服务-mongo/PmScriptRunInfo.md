# MongoDB 集合 — PmScriptRunInfo

> 数据库：pmweb_db 或 pmpc_db | 集合：`pmwebScriptRunInfo_00~99` / `PmpcScriptRunInfo_00~99`
> DAO：`PmScriptRunInfoDAOImpl` (implements `IPmScriptRunInfoDAO`，@Repository)
> 实体：`cn.testin.realweb.pojo.mongo.PmScriptRunInfo`

## 分片规则

`PmScriptRunInfo.table(taskid)` → `BasePojo.getTable(taskid, "pmwebScriptRunInfo" 或 "PmpcScriptRunInfo")`：

- taskId 末 2 位数字 → `_00` ~ `_99`
- taskId 以 `pt` 开头 → pmpc_db（`PmpcScriptRunInfo_XX`），否则 → pmweb_db（`pmwebScriptRunInfo_XX`）

详见 [存储总览](../../web-pc处理服务/02-技术架构/00-存储总览.md)。

## 用途

脚本运行记录：每个"设备 × 脚本"（子子任务）一条文档，记录执行状态、结果分类、错误信息、耗时、浏览器/客户端信息与脚本快照。任务组装时批量插入，执行中由上报结果驱动更新；报告页的脚本列表、结果分类统计（聚合）、重测记录查询都读它。

DAO 中 web/pc 差异集中在字段路径：web 走 `webScript.*` / `browserInfo.*`，pc 走 `pcScript.*` / `clientInfo.*`。

## 主要字段

| 字段 | 类型 | 说明 |
|------|------|------|
| taskid | String | 任务ID（分片键、查询主条件） |
| subtaskid | String | 子任务ID（设备/浏览器粒度） |
| subsubtaskid | String | 子子任务ID（脚本粒度，get/update 主条件） |
| projectid | Integer | 项目ID |
| scriptNo | Integer | 脚本No |
| execStatus | Integer | 执行状态 |
| networkType | String | 网络类型 |
| resultCategory | Integer | 结果分类（聚合统计的分组键） |
| errorCode | Integer | 错误码 |
| errorMsg | String | 错误信息（支持模糊查询） |
| startExecTime / finishTime | Long | 开始/完成时间 |
| execTime / totalTime | Long | 耗时/总耗时 |
| browserInfo | BrowserInfo | 浏览器信息（Web端，runInfo 包） |
| clientInfo | ClientInfo | PC客户端信息（PC端，runInfo 包） |
| webScript | ScriptInfo | 脚本信息（Web端，taskDetail 包） |
| pcScript | ScriptInfo | 脚本信息（PC端） |
| retestMark | Integer | 补测标识（1=补测数据） |
| retestStatus | Integer | 补测状态 |
| originalOrderNum | Integer | 原始编排顺序（多端使用） |
| repeatTestMark | Integer | 补测标记（0 正常 / 1 补测，前端展示用，AIO-1906） |
| errorCauseTypeId | Integer | 错误原因类型ID（自定义错误分析） |
| reportExecuteStatus | Integer | 报告执行状态（ReportExecuteStatusEnum） |
| customizeErrorMsg | String | 自定义错误信息 |
| status / updatetime | Integer/Long | 继承 BasePojo（status 用于批量忽略 STATUS_OFF） |

## 核心操作

| DAO 方法 | 操作 | 调用者 |
|----------|------|--------|
| batchInsert(list) | 批量插入 | TaskProcessServiceImpl（任务组装） |
| get(taskid, subsubtaskid) | 按 taskid+subsubtaskid 查单条 | ReportServiceImpl, TaskProcessServiceImpl, ScriptRunInfoServiceImpl, NoticeServiceImpl |
| list(taskid, conditionMap, sort, keywords) | 条件列表（subtaskid/subsubtaskid/orderNum/retestMark，默认按 orderNum+retryNum 升序） | ReportServiceImpl, TaskServiceImpl, TaskProcessServiceImpl, GenerateReportServiceImpl, ScriptSummaryServiceImpl, NoticeServiceImpl |
| update(scriptRunInfo) | 按 taskid+subsubtaskid 更新 execStatus/resultCategory/errorCode/errorMsg/时间/耗时/browserInfo/clientInfo/webScript/pcScript/retest*/errorCauseTypeId/reportExecuteStatus/customizeErrorMsg | ReportServiceImpl, TaskProcessServiceImpl, ScriptRunInfoServiceImpl, TaskServiceImpl, NoticeServiceImpl |
| baseList(taskid, conditionMap, sort, keywords, page, pageSize) | 分页查询，支持 systemError 解析、resultCategorys×errorCauseTypeIds OR 组合、脚本描述/设备信息模糊匹配、isAnalyze | ReportService（mvc）, ScriptRunInfoServiceImpl, NoticeServiceImpl, NewTaskService |
| batchIgnore(taskid, subtaskid) | 批量置 status=STATUS_OFF（忽略设备上全部脚本） | DeviceRunInfoServiceImpl |
| scriptCategorys(taskid, conditionMap, keywords) | 按条件查脚本分类相关字段 | TaskProcessServiceImpl, TaskServiceImpl, NoticeServiceImpl |
| aggregateCategory(taskid, conditionMap) | 聚合管道：match 后按 resultCategory 分组计数 | ScriptRunInfoServiceImpl |
| reTestRunInfo(runInfoQueryDTO) | 补测记录分页查询（retestMark=RE_TEST，可按 order/时间过滤） | 重测查询链路（RunInfoQueryDTO/RunInfoResult） |
| getDevicesByTaskId(taskId) | 聚合去重设备 type+version 组合 | ReportService（mvc，438 行） |

## 代码引用

| 调用者 | 场景 |
|--------|------|
| `TaskProcessServiceImpl` | 任务组装 batchInsert（521）、重测时读旧任务记录（get 380/405）、脚本状态维护（get 670、list 473/1001、update 1036、scriptCategorys 1076） |
| `ReportServiceImpl` | 上报结果更新脚本运行状态（get 208/341/757/1235/2230/2692/4047/4388、update 488/4080/4423、list 1059/1243/2957/4440） |
| `ScriptRunInfoServiceImpl` | 报告页脚本列表/详情/统计（baseList 115、aggregateCategory 152、get/update/list） |
| `DeviceRunInfoServiceImpl` | 设备忽略时级联 batchIgnore（232） |
| `ReportService`（mvc/service） | 报告列表分页 baseList（156/319）、设备列表 getDevicesByTaskId（438） |
| `NoticeServiceImpl` | 通知/邮件内容组装（baseList 580/1397/2573、list 2812/5508、scriptCategorys 5789、update 5811） |
| `TaskServiceImpl` | 任务维度脚本查询与状态修正（list 2937、scriptCategorys 3465/3504、update 3497/3520/3547） |
| `GenerateReportServiceImpl` | 报告生成时读取脚本运行记录（list 255/1673） |
| `NewTaskService` | 新任务链路 baseList（67） |
| `RealWebApi` / `McPcTaskApi` | 经 `IScriptRunInfoService` 间接调用（Web/PC 任务详情） |

## 相关文档

- [存储总览](../../web-pc处理服务/02-技术架构/00-存储总览.md)
- [PmTaskDetail](PmTaskDetail.md)
- [PmDeviceRunInfo](PmDeviceRunInfo.md)
- [PmReportDetail](PmReportDetail.md)
