# plan_task_template_config (db_plan)

- 用途：计划任务模板配置，存储任务模板相关的参数。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| plan_info_id | bigint(20) | 计划 ID |
| task_template_id | int(11) | 任务模板 ID |
| config_value | text | 配置值 |
| create_time | timestamp | 创建时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[PlanTaskController](../../平台基础功能服务/07-开放接口文档/测试计划/PlanTaskController.md)
