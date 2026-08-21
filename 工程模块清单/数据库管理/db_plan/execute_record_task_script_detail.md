# execute_record_task_script_detail (db_plan)

- 用途：每条脚本的详细执行信息，脚本执行完成后上报保存。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| execute_record_id | bigint(20) | 执行记录 ID |
| sub_sub_task_id | varchar(100) | 真正执行的子子任务 ID |
| execute_record_task_script_id | bigint(20) | 关联脚本 ID |
| device_id | varchar(255) | 设备 ID |
| script_id | int(11) | 脚本 ID |
| execute_result | int(11) | 执行结果 |
| result_category | int(11) | 失败原因分类 |
| error_cause_type_id | int(11) | 自定义错误类型 |
| execute_start_time | datetime(3) | 开始时间 |
| execute_end_time | datetime(3) | 结束时间 |
| execute_cost_time | bigint(20) | 执行总耗时 |
| script_init_cost_time | bigint(20) | 初始化耗时 |
| script_step_cost_time | bigint(20) | 步骤耗时 |
| pre_param | text | 执行前参数 |
| post_param | text | 执行后参数 |
| script_data_detail_uuid | varchar(255) | 数据行唯一标识 |
| row_id | int(11) | 数据行 ID |
| create_time | timestamp | 创建时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[ExecuteRecordTaskController](../../平台基础功能服务/07-开放接口文档/测试计划/ExecuteRecordTaskController.md)
