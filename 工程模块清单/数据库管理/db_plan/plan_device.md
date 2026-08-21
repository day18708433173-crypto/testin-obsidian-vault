# plan_device (db_plan)

- 用途：测试计划指定的设备列表，用于设备筛选和分配。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| plan_info_id | bigint(20) | 计划 ID |
| device_id | varchar(100) | 设备 ID |
| device_type | int(11) | 设备类型 |
| create_time | timestamp | 创建时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[PlanDeviceController](../../平台基础功能服务/07-开放接口文档/测试计划/PlanDeviceController.md)
