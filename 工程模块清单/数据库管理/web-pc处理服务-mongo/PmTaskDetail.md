# MongoDB 集合 — PmTaskDetail

> 数据库：pmweb_db 或 pmpc_db | 集合：`pmwebTaskDetail_00~99` / `PmpcTaskDetail_00~99`
> DAO：`PmTaskDetailDAOImpl` (implements `IPmTaskDetailDAO`)
> 实体：`cn.testin.realweb.pojo.mongo.PmTaskDetail`

## 分片规则

`BasePojo.getTable(taskid, "pmwebTaskDetail" 或 "PmpcTaskDetail")`：
- taskId 末 2 位数字 → `_00` ~ `_99`
- taskId 以 `pt` 开头 → pmpc_db（`PmpcTaskDetail_XX`），否则 → pmweb_db（`pmwebTaskDetail_XX`）

## 主要字段

| 字段 | 类型 | 说明 |
|------|------|------|
| taskid | String | 任务ID（主键级） |
| projectid | Integer | 项目ID |
| userid | Integer | 用户ID |
| userName | String | 用户名 |
| userEmail | String | 用户邮箱 |
| bizCode | String | 业务编码 |
| testType | String | 测试类型 |
| execStatus | String | 执行状态（init/ing/finish/cancel） |
| startExecTime | Long | 开始执行时间（毫秒时间戳） |
| finishTime | Long | 完成时间 |
| testResult | String | 测试结果 |
| execStandard | String | 执行标准（fast/normal/data） |
| scripts | JSONArray | 脚本列表 |
| browsers | JSONArray | 浏览器列表（Web端） |
| pcs | JSONArray | PC客户端列表（PC端） |
| retryNum | Integer | 重试次数 |
| level | Integer | 优先级（越小越高） |
| networks | JSONObject | 网络信息 |
| extendedChannel | String | 扩展渠道（如 TESTIN_TEST_PLAN） |
| content | JSONObject | 完整任务内容JSON |
| params | JSONObject | 参数信息 |
| status | Integer | 状态 |
| summarize | JSONObject | 汇总信息 |
| ciccDyeUrl | String | CICC染色URL |
| failNoticeScriptSendSign | Integer | 失败通知发送标记 |
| timeIntervalList | JSONArray | 时间段列表 |
| updatetime | Long | 更新时间 |

## 核心操作

| DAO 方法 | 操作 | 调用者 |
|----------|------|--------|
| get(taskId, keywords) | 按 taskid 查询（指定返回字段） | TaskService, NewTaskService, ReportService 多处 |
| insert(taskId, pmTaskDetail) | 插入 | ITaskService.create |
| update(pmTaskDetail) | 按 taskid + updatetime 条件更新 | TaskDetailServiceImpl, TaskProcessServiceImpl |
| 更新字段集 | scripts/pcs/browsers/execStatus/testResult/startExecTime/finishTime/status/summarize/params/content/systemParamStrategy/failNoticeScriptSendSign/timeIntervalList/ciccDyeUrl/ciccDyeTaskClearSign | 各业务方法 |

## 代码引用

| 调用者 | 场景 |
|--------|------|
| `TaskController.getTaskDetail` | [TaskController](TaskController.md) 查询详情 |
| `TaskController.sendTestPlanResult` | [TaskController](TaskController.md) 读取 extendedChannel |
| `ReportService.getNeedAnalysisScriptList` | [ProblemAnalysisReportController](ProblemAnalysisReportController.md) 读取 execStandard |
| `ITaskService.create` | 创建任务时写入 |
| `TaskProcessServiceImpl` | 状态机（init/complete）更新 execStatus 等 |
| `TaskDetailServiceImpl` | 更新 content/scripts/browsers/pcs |
| `McPcTaskApi.detail` | PC任务详情查询 |
| `RealWebApi.detail` | Web任务详情查询 |

## 相关文档

- [存储总览](../../web-pc处理服务/02-技术架构/00-存储总览.md)
- [TaskController](TaskController.md)
- [TestPlanController](TestPlanController.md)
