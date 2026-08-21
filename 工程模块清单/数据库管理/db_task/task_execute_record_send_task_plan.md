# task_execute_record_send_task_plan (db_task)

- 用途：需要发送给测试报告结果的数据会存入该表
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| task_execute_record_id | int | 任务id |
| task_execute_record_report_id | bigint | 报告id |
| task_execute_record_report_case_id | bigint | 报告caseId |
| task_execute_record_device_id | bigint | 设备id |
| create_time | datetime | 创建时间 |
| send_count | int | 发送次数 |
| type | int | 发送类型 |

- 关联接口：
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)
  - [核心链路-结果回收](../../任务管理服务/04-复杂功能细节/核心链路-结果回收.md)

- 关联表：
  - 主表 [task_execute_record](task_execute_record.md)
