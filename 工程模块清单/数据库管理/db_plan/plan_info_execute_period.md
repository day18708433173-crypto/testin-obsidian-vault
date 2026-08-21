# plan_info_execute_period (db_plan)

- 用途：测试计划的执行时间段配置，控制计划在哪些时间段内可执行。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| plan_info_id | bigint(20) | 计划 ID |
| start_time | varchar(100) | 开始时间 |
| end_time | varchar(100) | 结束时间 |
| type | int(11) | 时间段类型 |
| create_time | timestamp | 创建时间 |
| update_time | timestamp | 更新时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[PlanInfoExecutePeriodController](../../平台基础功能服务/07-开放接口文档/测试计划/PlanInfoExecutePeriodController.md)
