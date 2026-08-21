# plan_info_config (db_plan)

- 用途：计划信息配置，存储计划的额外配置参数。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| plan_info_id | bigint(20) | 计划 ID |
| config_key | varchar(255) | 配置键 |
| config_value | text | 配置值 |
| create_time | timestamp | 创建时间 |
| update_time | timestamp | 更新时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[PlanInfoConfigController](../../平台基础功能服务/07-开放接口文档/测试计划/PlanInfoConfigController.md)
