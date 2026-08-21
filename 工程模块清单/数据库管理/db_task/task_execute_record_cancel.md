# task_execute_record_cancel (db_task)

- 用途：任务取消记录
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| task_execute_record_id | int | 关联的任务表id |
| create_time | datetime | 创建时间 |
| cancel_count | int | 结果分析次数 |

- 关联接口：
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)

- 关联表：
  - 主表 [task_execute_record](task_execute_record.md)
