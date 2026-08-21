# plan_info_scheduled (db_plan)

- 用途：测试计划定时执行配置，支持 cron 表达式和手动设置。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| plan_info_id | bigint(20) | 测试计划 ID |
| scheduled_type | int(11) | 定时类型（1=cron 2=手动） |
| scheduled_rule | text | 定时规则 JSON |
| scheduled_cron | varchar(255) | cron 表达式 |
| strategy_type | int(11) | 执行冲突策略 |
| scheduled_status | int(11) | 启用状态 |
| create_user_id | int(11) | 创建人 |
| update_user_id | int(11) | 更新人 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[PlanInfoScheduledController](../../平台基础功能服务/07-开放接口文档/测试计划/PlanInfoScheduledController.md)
