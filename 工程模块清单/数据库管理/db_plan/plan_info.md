# plan_info (db_plan)

- 用途：测试计划主表，存储计划名称、类型、状态、项目关联等基本信息。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| project_id | int(11) | 项目 ID |
| plan_info_name | varchar(255) | 测试计划名称 |
| plan_info_type | int(11) | 计划类型 |
| plan_info_status | int(11) | 计划状态 |
| suite_id | int(11) | 关联应用 ID |
| test_stage | tinyint(1) | 测试阶段 |
| plan_device_status | tinyint(1) | 指定设备是否开启 |
| create_user_id | int(11) | 创建人 |
| update_user_id | int(11) | 更新人 |
| create_time | timestamp | 创建时间 |
| update_time | timestamp | 更新时间 |
| is_delete | varchar(100) | 是否删除 |

- 关联接口：[PlanInfoController](../../平台基础功能服务/07-开放接口文档/测试计划/PlanInfoController.md)
