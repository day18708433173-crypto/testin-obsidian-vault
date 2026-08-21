# MongoDB 集合 — PmDeviceRunInfo

> 数据库：pmweb_db 或 pmpc_db | 集合：`pmwebBrowserRunInfo_00~99`（Web）/ `PmpcClientRunInfo_00~99`（PC）
> DAO：`PmDeviceRunInfoDAOImpl` (implements `IPmDeviceRunInfoDAO`)
> 实体：`cn.testin.realweb.pojo.mongo.PmDeviceRunInfo`

## 分片规则

`PmDeviceRunInfo.table(taskid)` → `BasePojo.getTable(taskid, "pmwebBrowserRunInfo" 或 "PmpcClientRunInfo")`：

- taskId 末 2 位数字 → `_00` ~ `_99`
- taskId 以 `pt` 开头 → pmpc_db（`PmpcClientRunInfo_XX`），否则 → pmweb_db（`pmwebBrowserRunInfo_XX`）

注意：6 类集合中唯一 Web/PC 集合名不同构的一类（Browser vs Client）。详见 [存储总览](../../web-pc处理服务/02-技术架构/00-存储总览.md)。

## 用途

设备/浏览器运行记录：每个设备（子任务粒度）一条文档，记录该设备整体的执行状态、结果分类、错误信息、起止时间、浏览器/PC 客户端信息、数据驱动当前行。任务组装时批量插入，上报结果驱动状态收敛；报告页"按设备视角"列表与设备维度结果统计读它。

## 主要字段

| 字段 | 类型 | 说明 |
|------|------|------|
| taskid | String | 任务ID（分片键、查询主条件） |
| subtaskid | String | 子任务ID（设备粒度，get/update 主条件） |
| projectid | Integer | 项目ID |
| execStatus | Integer | 执行状态 |
| networkType | String | 网络类型 |
| resultCategory | Integer | 结果分类（聚合统计的分组键） |
| curExecSubsubtaskid | String | 当前正在执行的子子任务 |
| errorCode | Integer | 错误码 |
| errorCauseTypeId | Integer | 错误原因类型ID |
| errorMsg | String | 错误信息 |
| startExecTime / finishTime | Long | 开始/完成时间 |
| browserInfo | BrowserInfo | 浏览器信息（Web端，runInfo 包，字段：type/version/ip/osName） |
| clientInfo | ClientInfo | PC客户端信息（PC端，字段：systemVersion/systemName/systemType/ip 等） |
| retestMark | Integer | 补测标识（1=补测数据） |
| retestStatus | Integer | 补测状态 |
| dataRow | Integer | 当前子任务选择的行（仅数据驱动使用） |
| status / updatetime | Integer/Long | 继承 BasePojo |

## 核心操作

| DAO 方法 | 操作 | 调用者 |
|----------|------|--------|
| batchInsert(list) | 批量插入 | TaskProcessServiceImpl（任务组装，464 行） |
| get(taskid, subtaskid) | 按 taskid+subtaskid 查单条 | TaskServiceImpl, TaskProcessServiceImpl, ReportServiceImpl, DeviceRunInfoServiceImpl |
| getCountByTaskId(taskid) | 判断任务是否存在设备记录（count>0） | TaskServiceImpl（2795 行，bizCode=4200 校验） |
| list(taskid, conditionMap, keywords) | 条件列表（subtaskid/retestMark，按 orderNum 升序） | TaskServiceImpl, TaskProcessServiceImpl, TaskSummaryServiceImpl, GenerateReportServiceImpl, DeviceRunInfoServiceImpl |
| update(pmDeviceRunInfo) | 按 taskid+subtaskid 更新 execStatus/resultCategory/errorCode/errorMsg/时间/browserInfo/clientInfo/status/retest*/errorCauseTypeId | TaskServiceImpl, ReportServiceImpl, DeviceRunInfoServiceImpl |
| baseList(taskid, conditionMap, sort, keywords, page, pageSize) | 分页查询；web 按 browserInfo.type/version/ip/osName 过滤，pc 按 clientInfo.systemVersion/systemName/systemType/ip 过滤 | DeviceRunInfoServiceImpl |
| aggregateCategory(taskid, conditionMap) | 聚合管道：match 后按 resultCategory 分组计数（设备维度统计） | DeviceRunInfoServiceImpl |

## 代码引用

| 调用者 | 场景 |
|--------|------|
| `TaskProcessServiceImpl` | 任务组装 batchInsert（464）、重测/状态维护（list 448/1040、get 1028/1093） |
| `TaskServiceImpl` | 设备记录存在性校验 getCountByTaskId（2795）、设备信息补全 get（2817）、测试计划渠道校验 list（2836）、脚本/设备状态修正 list（2931/3566）、update（3591） |
| `ReportServiceImpl` | 上报结果收敛设备状态（get 267/499/755/1020/3065/3108/3129、update 507/519/4448） |
| `DeviceRunInfoServiceImpl` | 报告页设备列表/详情/统计：get（67/205）、update（87）、baseList（108/152）、aggregateCategory（120/164）、list（286） |
| `TaskSummaryServiceImpl` | 汇总统计读取设备运行记录（list 83） |
| `GenerateReportServiceImpl` | 报告生成时读取设备运行记录（list 246/1631） |

## 相关文档

- [存储总览](../../web-pc处理服务/02-技术架构/00-存储总览.md)
- [PmTaskDetail](PmTaskDetail.md)
- [PmScriptRunInfo](PmScriptRunInfo.md)
- [PmReportDetail](PmReportDetail.md)
