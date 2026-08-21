# plan_task_strategy (db_plan)

- 用途：测试计划任务的执行策略，控制串行/并行、失败后操作等。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| plan_info_id | bigint(20) | 关联测试计划 ID |
| sub_plan_info_id | bigint(20) | 关联子计划 ID |
| relation_task_type | int(11) | 关联任务类型（1=前置 2=中间 3=后置） |
| execute_order | int(11) | 执行顺序类型（1=串行 2=并行） |
| after_fail_operate | int(11) | 失败后操作（1=执行后置后终止 2=直接终止） |
| update_user_id | int(11) | 更新人 |
| create_time | timestamp | 创建时间 |
| update_time | timestamp | 更新时间 |
| is_delete | varchar(100) | 是否删除 |

- 关联接口：[PlanTaskStrategyController](../../平台基础功能服务/07-开放接口文档/测试计划/PlanTaskStrategyController.md)
