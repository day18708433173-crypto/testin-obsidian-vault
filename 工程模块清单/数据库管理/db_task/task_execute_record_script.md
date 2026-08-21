# task_execute_record_script (db_task)

- 用途：任务选择的脚本的快照
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| task_execute_record_id | int | 关联的任务id |
| script_type | int | 1为脚本，2为脚本组。打算把脚本组去掉 |
| script_execute_type | int | 1为app脚本，3为web脚本，5为pc脚本 |
| script_no | int | script_type为1表示关联的script_no，script_type为2表示关联的script_group_id |
| order_num | int | 表示执行数据顺序，task_type为1时有效 |
| count | int | 当前脚本执行次数 |
| cover_install | int | 执行前卸载安装.1为卸载安装，0为不卸载 |
| overwrite_install | int | 执行前覆盖安装，1为覆盖安装，0为不覆盖安装。和cover_install 字段只有一个能为1 |
| clean_data | int | 执行后清理数据。1为清理，0为不清理。 |
| keep_app | int | 执行后关闭应用，1为不关闭，0为关闭 |
| termination_on_error | int | 当前脚本执行错误，是否终止下边的脚本继续执行。0继续执行，1终止后续脚本 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |
| script_id | int | 触发时的script_id的快照 |
| script_name | varchar | 脚本名称 |
| script_tags | varchar | 脚本当时关联的tag |
| script_url | varchar | 脚本Url |
| script_md5 | varchar | 脚本Md5 |
| script_in_group | varchar | 脚本组下对应的脚本no关联的脚本id快照信息。脚本组去掉后，该字段无用 |
| task_execute_record_case_id | bigint |  |
| task_execute_record_case_step_id | bigint |  |
| case_id | int |  |
| case_step_id | int |  |
| case_step_order | int |  |
| pre_case_step_ids | varchar |  |
| aft_case_step_ids | varchar |  |
| step_expect | varchar |  |
| step_desc | varchar |  |

- 关联接口：
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)

- 关联表：
  - 主表 [task_execute_record](task_execute_record.md)
