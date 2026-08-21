# task_execute_result (db_portal)

- 用途：任务执行结果，存储真机任务的最终执行状态和统计信息。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| taskid | varchar(64) | 任务 ID |
| execute_status | int(11) | 执行状态 |
| execute_result | text | 执行结果 |
| total_count | int(11) | 总数 |
| success_count | int(11) | 成功数 |
| fail_count | int(11) | 失败数 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 关联接口：[portal-Task](../../平台基础功能服务/07-开放接口文档/任务管理/portal-Task.md)
