# plan_info_task_config (db_plan)

- 用途：测试计划与任务配置的关联表。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| plan_info_id | bigint(20) | 计划 ID |
| task_id | int(11) | 任务 ID |
| create_time | timestamp | 创建时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[PlanInfoTaskConfigController](../../平台基础功能服务/07-开放接口文档/测试计划/PlanInfoTaskConfigController.md)
