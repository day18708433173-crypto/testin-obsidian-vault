# execute_record_task_case_script_detail (db_plan)

- 用途：用例中每条脚本的详细执行结果，脚本执行完成后上报。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| execute_record_id | bigint(20) | 执行记录 ID |
| execute_record_task_case_id | bigint(20) | 用例记录 ID |
| script_id | int(11) | 脚本 ID |
| script_no | int(11) | 脚本编号 |
| execute_result | int(11) | 执行结果 |
| result_category | int(11) | 失败原因分类 |
| execute_start_time | datetime | 开始时间 |
| execute_end_time | datetime | 结束时间 |
| execute_cost_time | bigint(20) | 执行耗时 |
| create_time | timestamp | 创建时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[ExecuteRecordTaskController](../../平台基础功能服务/07-开放接口文档/测试计划/ExecuteRecordTaskController.md)
