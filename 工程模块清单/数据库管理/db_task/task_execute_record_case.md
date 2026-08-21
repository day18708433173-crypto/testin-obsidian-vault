# task_execute_record_case (db_task)

- 用途：执行记录-用例关联
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| task_execute_record_id | int | 关联的任务id |
| case_id | int | 用例ID |
| case_name | varchar | 用例名称 |
| order_num | int | 用例顺序 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 关联接口：
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)
  - [TaskExecuteRecordCaseController](../../任务管理服务/07-开放接口文档/用例报告/TaskExecuteRecordCaseController.md)、[TaskExecuteRecordCaseReportController](../../任务管理服务/07-开放接口文档/用例报告/TaskExecuteRecordCaseReportController.md)

- 关联表：
  - 主表 [task_execute_record](task_execute_record.md)
