# task_execute_record_notice (db_task)

- 用途：执行记录-通知配置
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 主键 |
| task_execute_record_id | int | 关联的任务id |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 关联接口：
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)

- 关联表：
  - 主表 [task_execute_record](task_execute_record.md)
