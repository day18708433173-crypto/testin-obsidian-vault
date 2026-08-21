# task_template_detail (db_task)

- 用途：模板执行详细配置
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 任务模板详情更新表 |
| task_template_id | int | 关联任务模板表的主键 |
| device_focus_scheduling | int | 设备配置集中调度 |
| env_id | int | 任务关联的环境id |
| app_package_id | int | app提测相关。提测选择的app包id |
| latest_package | int | app提测相关。是否使用最新的包，0为不使用，1为使用 |
| package_url | varchar | app提测相关。直接使用的包的url信息 |
| network_type | int | app提测相关，选择了app包有效。1为wifi。2为sim。3为模拟网络 |
| system_platform_id | int | app提测相关。包有关的系统平台id |
| simulate_network_name | varchar | 模拟网络名称，network_type为2时有效 |
| execute_method | int | 执行方式，1为分布式执行，2为顺序执行(普通执行) |
| data_source_id | int | 关联的数据源id |
| data_distribute_type | int | 数据下发类型。0为按设备分配，1为按脚本分配，2为数据驱动。0和1针对execute_type为2的情况。2针对execute_type为1的情况。 |
| data_tag | varchar | data_source_id存在时有效，表示需要选择的数据标签 |
| skip_data_tag | varchar | data_source_id存在时有效，表示需要跳过的数据 |
| finish_call_back_url | varchar | 任务完成的回调地址 |
| call_back_additional | varchar | 任务回调时需要增加返回的额外信息 |
| device_offline_config | int | 设备离线时配置 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 关联接口：
  - [TaskTemplateController](../../任务管理服务/07-开放接口文档/任务模板管理/TaskTemplateController.md)

- 关联表：
  - 主表 [task_template](task_template.md)
