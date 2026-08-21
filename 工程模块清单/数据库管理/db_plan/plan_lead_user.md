# plan_lead_user (db_plan)

- 用途：测试计划负责人信息，记录计划负责人和业务负责人。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| plan_info_id | bigint(20) | 关联测试计划 ID |
| lead_user_id | int(11) | 负责人 ID |
| lead_type | int(11) | 领导类型（1=计划负责人 2=业务负责人） |
| create_time | timestamp | 创建时间 |
| update_time | timestamp | 更新时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[PlanInfoController](../../平台基础功能服务/07-开放接口文档/测试计划/PlanInfoController.md)
