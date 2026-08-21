# plan_task_script (db_plan)

- 用途：计划任务关联的脚本信息（随用户模板修改而更新）。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| plan_info_id | bigint(20) | 计划 ID |
| sub_plan_info_id | bigint(20) | 子计划 ID |
| plan_task_id | bigint(20) | 计划任务 ID |
| script_no | int(11) | 脚本编号 |
| relation_task_type | int(11) | 任务类型 |
| status | int(11) | 状态 |
| create_time | timestamp | 创建时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[PlanTaskController](../../平台基础功能服务/07-开放接口文档/测试计划/PlanTaskController.md)
