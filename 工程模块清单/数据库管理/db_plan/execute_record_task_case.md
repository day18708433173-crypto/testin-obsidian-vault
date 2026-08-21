# execute_record_task_case (db_plan)

- 用途：执行记录中任务关联的测试用例快照。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| execute_record_id | bigint(20) | 执行记录 ID |
| execute_record_task_id | bigint(20) | 执行任务 ID |
| case_id | int(11) | 用例 ID |
| case_uuid | varchar(50) | 用例 UUID |
| case_name | varchar(255) | 用例名称 |
| case_dir_id | int(11) | 目录 ID |
| execute_status | int(11) | 执行状态 |
| create_time | timestamp | 创建时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[ExecuteRecordTaskController](../../平台基础功能服务/07-开放接口文档/测试计划/ExecuteRecordTaskController.md)
