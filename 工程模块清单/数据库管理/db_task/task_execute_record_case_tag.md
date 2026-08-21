# task_execute_record_case_tag (db_task)

- 用途：用例标签记录
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| task_execute_record_id | int | 关联的任务id |
| task_execute_record_case_id | int |  |
| case_id | int | 用例ID |
| case_name | varchar |  |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |
| tag_name | varchar |  |

- 关联接口：
  - [TaskExecuteRecordCaseController](../../任务管理服务/07-开放接口文档/用例报告/TaskExecuteRecordCaseController.md)、[TaskExecuteRecordCaseReportController](../../任务管理服务/07-开放接口文档/用例报告/TaskExecuteRecordCaseReportController.md)

- 关联表：
  - 主表 [task_execute_record_case](task_execute_record_case.md)
