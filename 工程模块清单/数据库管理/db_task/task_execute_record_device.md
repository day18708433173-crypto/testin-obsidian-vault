# task_execute_record_device (db_task)

- 用途：关联的设备表，以及设备的基础信息快照
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 模板关联的设备信息 |
| task_execute_record_id | int | 关联的任务id |
| device_id | varchar | task_type为1，设备id，App返回的deviceId；task_type为3时，Web返回中的'ip_osName_type_version'字段组合；task_type为5时，桌面返回中的desktopId字段 |
| device_source | varchar | 设备云id |
| ucom_id | varchar | 上位机id |
| device_type | int | 设备类型，设备类型，1为app，3为web，5为pc |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |
| device_status | int | 设备的执行状态 |
| brand_name | varchar | 品牌名称 |
| alias_name | varchar | 设备别名 |
| dpi_width | int | 屏幕分辨率宽度 |
| dpi_height | int | 屏幕分辨率高度 |
| serial_number | varchar | 设备序列号 |
| system_platform_name | varchar | 设备平台名称 |
| web_device_type_name | varchar | web执行的浏览器类型名称 |
| web_version | varchar | web浏览器的版本 |
| os_name | varchar | web浏览器的操作系统 |
| task_execute_record_case_step_id | int | 用例执行步骤id |
| ucom_ip | varchar |  |
| release_version | varchar |  |
| descr | varchar |  |
| location | varchar |  |
| model_name | varchar |  |

- 关联接口：
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)

- 关联表：
  - 主表 [task_execute_record](task_execute_record.md)
