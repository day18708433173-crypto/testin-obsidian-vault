# task_execute_record_time_period (db_task)

- 用途：存放任务下发时间段的表
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 自增id |
| task_execute_record_id | int | 关联的任务id |
| start_time | bigint | 开始时间 |
| end_time | bigint | 结束时间 |
| type | int | 类型：1不下发直接暂停任务，2时间区间内下发任务 |
| create_time | datetime | 创建时间 |

- 关联接口：
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)

- 关联表：
  - 主表 [task_execute_record](task_execute_record.md)
