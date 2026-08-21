# task_execute_record_report_case (db_task)

- 用途：执行报告-用例结果表。记录每个用例在任务执行中的详细结果，包括错误类型、缺陷平台ID、重试次数等。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 主键，自增 |
| task_execute_record_id | int | 关联执行记录ID |
| task_execute_record_report_id | bigint | 关联报告ID |
| case_id | int | 用例ID |
| case_name | varchar | 用例名称 |
| case_dir_name | varchar | 用例目录路径 |
| script_id | int | 关联脚本ID |
| device_id | int | 执行设备ID |
| case_status | int | 用例执行状态 |
| error_type_id | int | 错误类型ID |
| error_cause_type_id | int | 错误原因类型ID |
| error_description | text | 错误描述 |
| defect_platform_id | varchar | 缺陷平台关联ID |
| retry_count | int | 重试次数 |
| execute_time | bigint | 执行耗时（毫秒） |
| input_param | text | 执行输入参数（JSON） |
| error_images | text | 错误截图URL列表 |
| start_time | timestamp | 执行开始时间 |
| end_time | timestamp | 执行结束时间 |
| create_time | timestamp | 创建时间 |

- 关联接口：
  - [TaskExecuteRecordCaseController](../../任务管理服务/07-开放接口文档/用例报告/TaskExecuteRecordCaseController.md)
  - [TaskExecuteRecordCaseReportController](../../任务管理服务/07-开放接口文档/用例报告/TaskExecuteRecordCaseReportController.md)

- 关联子表：
  - `task_execute_record_case_step` — 用例步骤执行明细
