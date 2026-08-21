# task_execute_record_result_analysis (db_task)

- 用途：任务结果解析后，要对结果进行分析会进入该表
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| task_execute_record_id | int | 关联的任务表id |
| task_execute_record_report_id | bigint | 报告id |
| retry_num | int | 重试次数，为0表示最后一次数据。 |
| create_time | datetime | 创建时间 |
| result_url | varchar | 需要解析的地址 |
| analysis_count | int | 结果分析次数 |

- 关联接口：
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)
  - [核心链路-结果回收](../../任务管理服务/04-复杂功能细节/核心链路-结果回收.md)

- 关联表：
  - 主表 [task_execute_record](task_execute_record.md)
