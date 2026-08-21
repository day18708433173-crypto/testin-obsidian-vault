# task_template_device (db_task)

- 用途：模板关联的设备表
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 模板关联的设备信息 |
| task_template_id | int | 关联的任务模板id |
| device_id | varchar | task_type为1，设备id，App返回的deviceId；task_type为3时，Web返回中的'ip_osName_type_version'字段组合；task_type为5时，桌面返回中的desktopId字段 |
| device_name | varchar | 设备名称 |
| device_source | varchar | 设备云id |
| ucom_id | varchar | 上位机id |
| device_type | int | 设备类型，设备类型，1为app，3为web，5为pc |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 关联接口：
  - [TaskTemplateController](../../任务管理服务/07-开放接口文档/任务模板管理/TaskTemplateController.md)

- 关联表：
  - 主表 [task_template](task_template.md)
