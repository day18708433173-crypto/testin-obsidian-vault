# plan_info_task_template_status_sync (db_plan)

- 用途：任务模板状态同步记录，跟踪模板状态变更的同步情况。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| task_template_id | int(11) | 任务模板 ID |
| sync_status | int(11) | 同步状态 |
| create_time | timestamp | 创建时间 |

- 关联接口：[PlanInfoTaskConfigController](../../平台基础功能服务/07-开放接口文档/测试计划/PlanInfoTaskConfigController.md)
