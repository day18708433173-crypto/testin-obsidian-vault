# execute_record_task (db_plan)

- 用途：执行记录中的任务详情，记录每个任务/子任务/重测任务的执行快照和统计。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| execute_record_id | bigint(20) | 执行记录 ID |
| plan_info_id | bigint(20) | 测试计划 ID |
| sub_plan_info_id | bigint(20) | 子计划 ID |
| plan_task_id | bigint(20) | 计划任务 ID |
| relation_task_id | int(11) | 关联任务 ID |
| execute_task_id | varchar(100) | 实际执行任务 ID |
| record_task_type | int(11) | 记录类型（1=子任务 2=任务 3=重测） |
| execute_status | int(11) | 执行状态 |
| relation_task_type | int(11) | 任务类型（1=前置 2=列表 3=后置） |
| parent_id | bigint(20) | 父节点 |
| order_num | int(11) | 排序 |
| execute_start_time | timestamp | 执行开始 |
| execute_end_time | timestamp | 执行结束 |
| task_total | int(11) | 任务总量 |
| complete_task_count | int(11) | 完成数 |
| script_total | int(11) | 脚本总数 |
| device_total | int(11) | 设备总数 |
| success_script_count | int(11) | 成功脚本数 |
| fail_script_count | int(11) | 失败脚本数 |
| execute_cost_time | bigint(20) | 执行耗时 |
| case_num | int(11) | 用例数 |
| success_case_count | int(11) | 通过用例数 |
| fail_case_count | int(11) | 失败用例数 |
| creating_time | timestamp | 创建时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[ExecuteRecordTaskController](../../平台基础功能服务/07-开放接口文档/测试计划/ExecuteRecordTaskController.md)
