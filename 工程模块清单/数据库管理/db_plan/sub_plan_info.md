# sub_plan_info (db_plan)

- 用途：子计划信息，表示测试计划下的执行子阶段（如功能测试阶段、回归测试阶段）。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| plan_info_id | bigint(20) | 关联测试计划 ID |
| sub_plan_name | varchar(255) | 子计划名称 |
| order_num | int(11) | 排序号 |
| execute_time | varchar(100) | 执行触发时间点 HH:mm:ss |
| parallel_priority | int(11) | 插队优先级 |
| create_user_id | int(11) | 创建人 |
| update_user_id | int(11) | 更新人 |
| create_time | timestamp | 创建时间 |
| update_time | timestamp | 更新时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[SubPlanInfoController](../../平台基础功能服务/07-开放接口文档/测试计划/SubPlanInfoController.md)
