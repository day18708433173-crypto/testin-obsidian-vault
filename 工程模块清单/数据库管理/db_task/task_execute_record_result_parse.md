# task_execute_record_result_parse (db_task)

- 用途：等待解析结果的数据会存入该表
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| task_execute_record_id | int | 关联的执行任务id |
| task_execute_record_report_id | bigint | 关联的执行任务的报告id |
| retry_num | int | 重试次数，为0表示最后一次数据。 |
| create_time | datetime | 记录创建时间 |
| result_url | varchar | 需要解析的地址 |
| parse_count | int | 获取解析的次数 |

- 关联接口：
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)
  - [核心链路-结果回收](../../任务管理服务/04-复杂功能细节/核心链路-结果回收.md)

- 关联表：
  - 主表 [task_execute_record](task_execute_record.md)
