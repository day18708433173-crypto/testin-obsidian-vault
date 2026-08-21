# task_execute_record_type (db_task)

- 用途：执行记录类型
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| task_execute_record_id | int | 用例执行记录id |
| task_execute_record_type | int | 用例执行记录 1-app 3-web 5-pc |

- 关联接口：
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)

- 关联表：
  - 主表 [task_execute_record](task_execute_record.md)
