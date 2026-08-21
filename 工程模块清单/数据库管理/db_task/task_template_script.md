# task_template_script (db_task)

- 用途：任务模板关联的scriptNo
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| task_template_id | int | 关联的任务模板id |
| script_type | int | 1为脚本，2为脚本组。打算把脚本组去掉 |
| script_no | int | script_type为1表示关联的script_no，script_type为2表示关联的script_group_id |
| script_execute_type | int | 1为app脚本，3为web脚本，5为pc脚本 |
| order_num | int | 表示执行数据顺序，task_type为1时有效 |
| count | int | 当前脚本执行次数 |
| cover_intasll | int | 执行前卸载安装.1为卸载安装，0为不卸载 |
| overwrite_install | int | 执行前覆盖安装，1为覆盖安装，0为不覆盖安装。和cover_install 字段只有一个能为1 |
| clean_data | int | 执行后清理数据。1为清理，0为不清理。 |
| keep_app | int | 执行后关闭应用，1为不关闭，0为关闭 |
| termination_on_error | int | 当前脚本执行错误，是否终止下边的脚本继续执行。0继续执行，1终止后续脚本 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 关联接口：
  - [TaskTemplateController](../../任务管理服务/07-开放接口文档/任务模板管理/TaskTemplateController.md)

- 关联表：
  - 主表 [task_template](task_template.md)
