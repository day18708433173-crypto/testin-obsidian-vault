# plan_task (db_plan)

- 用途：计划关联的任务，记录计划中引用的具体测试任务信息。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| plan_info_id | bigint(20) | 关联测试计划 ID |
| sub_plan_info_id | bigint(20) | 关联子计划 ID |
| relation_task_type | int(11) | 任务类型（1=前置 2=中间 3=后置） |
| relation_task_id | int(11) | 关联任务 ID |
| relation_task_name | varchar(255) | 关联任务名称 |
| relation_suite_id | int(11) | 关联任务应用 ID |
| relation_device_count | int(11) | 关联设备数量 |
| relation_script_count | int(11) | 关联脚本数量 |
| order_num | int(11) | 执行顺序 |
| create_user_id | int(11) | 创建人 |
| update_user_id | int(11) | 更新人 |
| create_time | timestamp | 创建时间 |
| update_time | timestamp | 更新时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[PlanTaskController](../../平台基础功能服务/07-开放接口文档/测试计划/PlanTaskController.md)
