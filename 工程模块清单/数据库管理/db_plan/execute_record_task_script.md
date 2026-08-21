# execute_record_task_script (db_plan)

- 用途：执行记录中任务关联的脚本快照（去重）。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| execute_record_id | bigint(20) | 执行记录 ID |
| root_sub_plan_record_id | bigint(20) | 根子计划 ID |
| sub_plan_record_id | bigint(20) | 子计划记录 ID |
| task_and_reset_record_id | bigint(20) | 任务/重测记录 ID |
| execute_record_task_id | bigint(20) | 执行任务 ID |
| script_no | int(11) | 脚本编号 |
| relation_task_type | int(11) | 任务类型 |
| create_time | timestamp | 创建时间 |
| update_time | timestamp | 更新时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[ExecuteRecordTaskController](../../平台基础功能服务/07-开放接口文档/测试计划/ExecuteRecordTaskController.md)
