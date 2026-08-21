# task_execute_record_case_step (db_task)

- 用途：用例步骤执行记录
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| task_execute_record_id | int | 关联的任务id |
| task_execute_record_case_id | int |  |
| case_id | int | 用例ID |
| case_name | varchar |  |
| order_num | int |  |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |
| case_step_id | int |  |
| step_order | int |  |
| step_desc | varchar |  |
| step_expect | varchar |  |
| script_no | int |  |
| script_type | int |  |
| script_name | varchar |  |
| parallel_order | int |  |
| execute_status | int |  |
| test_result | int |  |
| cost_time | bigint |  |
| result_category | int |  |
| error_code_id | int |  |
| error_code_name | varchar |  |
| error_message | varchar |  |
| video_url | varchar |  |
| execute_end_time | datetime |  |

- 关联接口：
  - [TaskExecuteRecordCaseController](../../任务管理服务/07-开放接口文档/用例报告/TaskExecuteRecordCaseController.md)、[TaskExecuteRecordCaseReportController](../../任务管理服务/07-开放接口文档/用例报告/TaskExecuteRecordCaseReportController.md)

- 关联表：
  - 主表 [task_execute_record_report_case](task_execute_record_report_case.md)
