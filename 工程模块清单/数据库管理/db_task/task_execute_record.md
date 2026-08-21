# task_execute_record (db_task)

- 用途：任务执行记录主表。每次执行模板（手动/定时/计划/重测）时创建一条记录，追踪任务执行的完整生命周期。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 主键，自增 |
| project_id | int | 项目ID |
| task_type | int | 任务类型：1=App，3=Web，5=PC，1000=用例驱动 |
| suite_id | int | 关联应用ID |
| task_name | varchar | 任务名称 |
| task_status | int | 任务状态（TaskStatusEnum） |
| task_source | int | 任务来源：1=手动，2=定时任务，3=测试计划，4=重测 |
| execute_record_task_id | bigint | 关联测试计划任务ID（taskSource=3时） |
| execute_record_task_name | varchar | 测试计划任务名称 |
| execute_record_id | bigint | 关联测试计划执行记录ID |
| task_template_id | int | 来源模板ID（无关联模板时为0） |
| create_user_id | int | 创建人 |
| update_user_id | int | 更新人 |
| create_time | timestamp | 创建时间 |
| update_time | timestamp | 更新时间 |
| task_execute_id | varchar | 任务执行UUID |
| parent_id | int | 父任务ID（重测链追溯） |
| script_total | int | 脚本总数 |
| execute_script_total | int | 实际执行脚本数 |
| success_script_total | int | 成功脚本数 |
| fail_script_total | int | 失败脚本数 |
| skip_script_total | int | 跳过脚本数 |
| cancel_script_total | int | 取消脚本数 |
| timeout_script_total | int | 超时脚本数 |
| case_total | int | 用例总数 |
| execute_case_total | int | 实际执行用例数 |
| success_case_total | int | 通过用例数 |
| fail_case_total | int | 失败用例数 |
| skip_case_total | int | 跳过用例数 |
| cancel_case_total | int | 取消用例数 |
| timeout_case_total | int | 超时用例数 |
| device_total | int | 设备总数 |
| effective_execute_time | bigint | 有效执行时间（毫秒） |
| error_message | varchar | 执行异常错误信息 |
| end_time | timestamp | 任务结束时间 |

- 关联接口：
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)

- 关联子表：
  - `task_execute_record_detail` — 执行配置详情
  - `task_execute_record_script` — 脚本执行记录
  - `task_execute_record_device` — 设备执行记录
  - `task_execute_record_case` — 用例执行记录
  - `task_execute_record_report` — 执行报告
