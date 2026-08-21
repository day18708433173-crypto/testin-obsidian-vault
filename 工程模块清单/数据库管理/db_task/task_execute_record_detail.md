# task_execute_record_detail (db_task)

- 用途：执行记录详细配置
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 任务模板详情更新表 |
| task_execute_record_id | int | 关联任务表的主键 |
| device_focus_scheduling | int | 设备配置集中调度 |
| env_id | int | 任务关联的环境id |
| env_config | varchar | 环境快照信息 |
| level | int | 任务执行等级，越小优先级越高 |
| app_package_id | int | app提测相关。提测选择的app包id |
| latest_package | int | app提测相关。是否使用最新的包，0为不使用，1为使用 |
| package_url | varchar | app提测相关。直接使用的包的url信息 |
| network_type | int | app提测相关，选择了app包有效。1为wifi。2为sim。3为模拟网络 |
| system_platform_id | int | app提测相关。包有关的系统平台id |
| app_name | varchar | app提测相关，app名称 |
| app_package_name | varchar | app提测相关，包名称 |
| app_version | varchar | app提测相关，app版本 |
| app_size | bigint | app提测相关，app提测大小 |
| app_build | varchar | app版本号 |
| app_icon_url | varchar | app图标地址 |
| app_start_path | varchar | app的启动路径 |
| app_md5 | varchar | app的md5信息 |
| simulate_network_name | varchar | 模拟网络名称，network_type为2时有效 |
| network_config | varchar | 模拟网络的查询信息 |
| execute_method | int | 执行方式，1为分布式执行，2为顺序执行(普通执行) |
| old_task_id | int | 重测来源哪条数据，老的任务id |
| retest_status | varchar | 重测老的数据中，需要重测哪些状态的数据 |
| retest_report_ids | varchar | 重测老的数据中，需要重测的报告id |
| data_source_id | int | 关联的数据源id |
| data_distribute_type | int | 数据下发类型。0为按设备分配，1为按脚本分配，2为数据驱动。0和1针对execute_type为2的情况。2针对execute_type为1的情况。 |
| data_tag | varchar | data_source_id存在时有效，表示需要选择的数据标签 |
| skip_data_tag | varchar | data_source_id存在时有效，表示需要跳过的数据 |
| finish_call_back_url | varchar | 任务完成的回调地址 |
| call_back_additional | varchar | 任务回调时需要增加返回的额外信息 |
| device_offline_config | int | 设备离线配置 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |
| retest_case_ids | varchar |  |
| version_remark | varchar | 应用包描述 |

- 关联接口：
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)

- 关联表：
  - 主表 [task_execute_record](task_execute_record.md)
