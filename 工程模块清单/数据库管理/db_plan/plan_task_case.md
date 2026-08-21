# plan_task_case (db_plan)

- 用途：计划任务关联的测试用例信息。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| plan_info_id | bigint(20) | 计划 ID |
| sub_plan_info_id | bigint(20) | 子计划 ID |
| plan_task_id | bigint(20) | 计划任务 ID |
| case_id | int(11) | 用例 ID |
| status | int(11) | 状态 |
| create_time | timestamp | 创建时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[PlanTaskController](../../平台基础功能服务/07-开放接口文档/测试计划/PlanTaskController.md)
