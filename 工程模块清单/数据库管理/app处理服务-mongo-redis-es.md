---
branch: syy.release.z7.8.1.0
module: real-test
type: SQL
storage: MongoDB / Redis / Elasticsearch
---

# app处理服务 非关系型存储总览

app处理服务 模块使用 MySQL 存储任务配置/关系数据，使用 MongoDB 存储执行结果/详情（大 JSON 文档），Redis 缓存实时状态，Elasticsearch 做报告检索。

## MongoDB 集合

| 集合名 | DAO | 说明 | 使用场景 |
| --- | --- | --- | --- |
| pmreal_adapt_detail | PmrealAdaptDetailDAOImpl | 适配任务详情，含脚本列表/设备列表/参数/报告摘要 | Task.add/detail、全部任务操作 |
| pmreal_task_summary | PmrealTaskSummaryDAOImpl | 任务执行摘要(按子任务聚合) | 任务进度/汇总查询 |
| pmreal_report_detail | PmrealReportDetailDAOImpl | 报告步骤详情(每步状态/截图/日志/性能) | Report.stepinfos/stepdetail/scriptSteps |
| pmreal_script_summary | PmrealScriptSummaryDAOImpl | 脚本执行摘要(按脚本聚合) | Report.scriptsummaries、ScriptList.listScript |
| pmreal_device_detail | PmrealDeviceDetailDAOImpl | 设备执行详情(含性能数据) | Report.listDeviceReport、Performance.reportGraph |
| pmreal_stat_summary | PmrealStatSummaryDAOImpl | 任务统计摘要(设备分布/性能/抽查) | Task.getRealStatSummaryDetail、Report.getStatSummary |
| pmreal_spot_test_summary | PmrealSpotTestSummaryDAOImpl | 抽查测试摘要 | 质检抽查功能 |
| pmreal_stat_exception | PmrealStatExceptionDAOImpl | 异常统计 | Stat.exceptions、异常分析 |
| pmreal_task_run_info | PmrealTaskRunInfoDAOImpl | 任务实时运行状态 | 后台线程更新任务进度 |
| pmreal_data_run_info | PmrealDataRunInfoDAOImpl | 数据运行信息 | 脚本结果数据 |
| pmreal_check_info | PmrealCheckInfoDAOImpl | 脚本检查/检测信息 | ScriptReport.checkInfos |
| pmreal_error_summary | PmrealErrorSummaryDAOImpl | 错误摘要统计 | 错误分析报表 |
| pmreal_retest_summary | PmrealRetestSummaryDAOImpl | 补测摘要记录 | 补测历史查询 |
| pmreal_adapt_result | PmrealAdaptResultDAOImpl | 适配最终结果 | 适配完成回调 |
| pmreal_adapt_detail (多个) | PmrealAdaptDetailDAOImpl | 适配详情扩展集合 | 按需存储大字段 |

## Redis 键

| 键模式 | DAO | 说明 |
| --- | --- | --- |
| task_run_info:* | TaskRunInfoDAOImpl | 任务实时运行状态(设备/脚本在线状态) |
| data_run_info:* | DataRunInfoDAOImpl | 数据运行信息缓存 |
| script_collect:* | ScriptCollectDAOImpl | 脚本收集结果缓存 |
| script_collect_queue:* | ScriptCollectQueueDAOImpl | 脚本收集队列 |
| init_task:{eid}:{reqId} | InitTaskDAOImpl | 无App快速提测初始化任务 |

## Elasticsearch 索引

| 索引 | DAO | 说明 | 使用场景 |
| --- | --- | --- | --- |
| report_summary | EsReportSummaryDAOImpl | 报告摘要(按脚本/设备维度，含错误信息) | Report.list/reportList/summary、AnalysisReport.list |

## app处理服务 中的关键读写

### 写入链（以 TestResult.report 为例）
1. MongoDB: `pmreal_report_detail` (INSERT 步骤详情)
2. MongoDB: `pmreal_script_summary` (UPDATE 脚本状态)
3. MongoDB: `pmreal_task_summary` (UPDATE 任务进度)
4. ES: `report_summary` (INDEX 报告摘要)

### 读取链（以 Report.list 为例）
1. 优先 ES `report_summary` 查询（全文搜索+聚合）
2. 补充 MongoDB `pmreal_report_detail`（步骤级详情）
3. 补充 MongoDB `pmreal_script_summary`（脚本摘要）
4. 补充 MySQL `preal_user_adapt`（任务基本信息）
