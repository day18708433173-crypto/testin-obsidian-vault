# task_execute_record_report (db_task)

- 用途：执行报告主表
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| task_execute_record_id | int | 关联的任务id |
| data_identifier_id | varchar | 标识当前脚本的数据标识， |
| task_execute_record_script_id | bigint | 关联的脚本信息id |
| task_execute_record_device_id | bigint | 关联的设备信息id。该字段可能没有值。因为补测可以重新选设备，不一定能关联上（或者补测在对应的表新增一条数据） |
| script_no | int | 具体执行的脚本No，执行脚本时的冗余字段，执行脚本组时具体执行的脚本no |
| device_id | varchar | 执行的设备id |
| order_num | int | 执行顺序 |
| row_id | int | 执行数据行id |
| dependency_row_index | int | 依赖脚本有数据源时获取数据源的下标信息 |
| data_source_config_id | int | 数据源id |
| expire_time | bigint | 过期时间 |
| execute_status | int | 执行状态 |
| result_category | int | 执行结果分类，脚本失败的具体原因字段，成功时该字段为1 |
| error_message | varchar | 执行失败时的错误信息 |
| result_url | varchar | 上报结果的Url，test2结尾 |
| execute_start_time | datetime | 脚本执行的开始时间 |
| execute_end_time | datetime | 脚本执行的结束时间 |
| execute_cost_time | bigint | 执行总花费时间 |
| script_init_cost_time | bigint | 脚本初始化花费时间 |
| script_step_cost_time | bigint | 脚本步骤花费时间 |
| script_preparation_cost_time | bigint | 脚本准备阶段花费时间 |
| result_log_url | varchar | 结果统计中的日志url |
| old_data | int | 补测的老数据标识，重测需要将原数据标识为老数据。0为最新一条数据，1为老数据 |
| retest_num | int | 补测次数，正常数据为0，补测后在老数据上+1 |
| error_code_id | int | 关联的错误类型的id,由用户手动填写的信息 |
| error_code_name | varchar | 关联的错误类型名称,由用户手动填写的信息 |
| match_time | datetime | 匹配设备时间 |
| match_count | int | 匹配次数 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |
| device_cpu | int | 执行时，cpu的占用率 |
| device_memory | int | 执行时，内存使用率 |
| device_flow | int | 执行时的流量消耗 |
| device_temperature | int | 执行时设备温度 |
| task_record_report_case_id | bigint |  |
| case_id | int |  |
| case_step_id | int |  |
| case_step_order | int |  |
| pre_case_step_ids | varchar |  |
| aft_case_step_ids | varchar |  |
| script_execute_type | int |  |
| case_name | varchar |  |
| script_name | varchar |  |
| step_expect | varchar |  |
| step_desc | varchar |  |
| execute_result_summary | int | 执行结果汇总 |

- 关联接口：
  - [TaskExecuteRecordReportController](../../任务管理服务/07-开放接口文档/任务报告/TaskExecuteRecordReportController.md)

- 关联表：
  - 主表 [task_execute_record](task_execute_record.md)
