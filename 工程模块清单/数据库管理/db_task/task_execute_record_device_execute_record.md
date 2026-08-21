# task_execute_record_device_execute_record (db_task)

- 用途：设备执行记录
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| device_id | varchar | 设备id |
| ucom_id | varchar | 上位机id |
| device_type | int | 设备类型 |
| task_execute_record_report_case_id | bigint | 报告用例id |
| create_time | datetime | 设备占用时间 |
| task_execute_record_id | int | 执行记录id |
| task_execute_record_report_id | bigint | 执行报告id |

- 关联接口：
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)

- 关联表：
  - 主表 [task_execute_record](task_execute_record.md)
