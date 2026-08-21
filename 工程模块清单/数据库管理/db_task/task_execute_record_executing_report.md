# task_execute_record_executing_report (db_task)

- 用途：执行中报告
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| task_execute_record_id | int | 任务执行记录id |
| task_template_id | int | 任务模板id |
| task_execute_record_report_case_id | bigint | 用例报告id |
| task_execute_record_report_id | bigint | 执行记录id |
| status | int | 当前状态，1占用设备，0未占用设备 |
| create_time | bigint | 创建时间 |

- 关联接口：
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)

- 关联表：
  - 主表 [task_execute_record](task_execute_record.md)
