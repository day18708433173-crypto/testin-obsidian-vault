# MongoDB 集合 — PmTaskSummary

> 数据库：pmweb_db 或 pmpc_db | 集合：`pmwebTaskSummary_00~99` / `PmpcTaskSummary_00~99`
> DAO：`PmTaskSummaryDAOImpl` (implements `IPmTaskSummaryDAO`)
> 实体：`cn.testin.realweb.pojo.mongo.PmTaskSummary`

## 分片规则

`PmTaskSummary.table(taskid)` → `BasePojo.getTable(taskid, "pmwebTaskSummary" 或 "PmpcTaskSummary")`：

- taskId 末 2 位数字 → `_00` ~ `_99`
- taskId 以 `pt` 开头 → pmpc_db（`PmpcTaskSummary_XX`），否则 → pmweb_db（`pmwebTaskSummary_XX`）

详见 [存储总览](../../web-pc处理服务/02-技术架构/00-存储总览.md)。

## 用途

任务级报告统计汇总：脚本分类统计、浏览器/PC 问题分布、操作系统分布。每个任务一条文档，随报告生成过程增量更新（`update` 按 web/pc 前缀分别写入 browser* 或 pc* 字段）。

## 主要字段

| 字段 | 类型 | 说明 |
|------|------|------|
| taskid | String | 任务ID（查询/更新主条件） |
| vhost | Integer | 模块id（服务号） |
| eid | Integer | 企业ID |
| projectid | Integer | 项目ID |
| scriptCategorySummary | List\<CategorySummaryInfo\> | 脚本分类统计 |
| browserProblemSummary | List\<ProblemSummaryInfo\> | 浏览器问题分布统计（Web端） |
| pcProblemSummary | List\<ProblemSummaryInfo\> | 问题分布统计（PC端） |
| pcSystemSummary | List\<SystemSummaryInfo\> | 操作系统分布（PC端） |
| browserSystemSummary | List\<SystemSummaryInfo\> | 操作系统分布（Web端） |
| summarytime | Long | 最后统计的时间 |
| status | Integer | 状态（继承 BasePojo） |
| updatetime | Long | 更新时间（继承 BasePojo，update 时自动刷新） |

子结构位于 `cn.testin.realweb.pojo.mongo.statSummary` 包：`CategorySummaryInfo`、`ProblemSummaryInfo`、`SystemSummaryInfo`。

## 核心操作

| DAO 方法 | 操作 | 调用者 |
|----------|------|--------|
| get(taskid) | 按 taskid 查询单条 | TaskSummaryServiceImpl |
| insert(taskSummary) | 插入（首次统计时） | TaskSummaryServiceImpl |
| update(taskSummary) | 按 taskid 条件更新；web 前缀(wt)只更新 browser* 字段，pc 前缀(pt)只更新 pc* 字段，scriptCategorySummary 两端通用 | TaskSummaryServiceImpl |

## 代码引用

| 调用者 | 场景 |
|--------|------|
| `TaskSummaryServiceImpl` | 汇总统计的生成/读取：get 判断是否存在，不存在 insert，存在 update（约 256/263/287/304 行） |
| `GenerateReportServiceImpl` | 报告生成链路中经 `ITaskSummaryService` 触发汇总统计（约 238/1638 行） |
| `NoticeServiceImpl` | 多处经 `SpringHelper.getBean("ITaskSummaryService")` 读取汇总用于邮件/通知内容（REPORT_STAT 等） |
| `GenericBaseService.istatsummaryservice` | 基类注入，供各 Service 使用 |

## 相关文档

- [存储总览](../../web-pc处理服务/02-技术架构/00-存储总览.md)
- [PmTaskDetail](PmTaskDetail.md)
- [PmReportDetail](PmReportDetail.md)
- [PmScriptSummary](PmScriptSummary.md)
