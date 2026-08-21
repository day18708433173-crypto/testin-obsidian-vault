# task_template (db_task)

- 用途：任务模板主表，存储可复用的任务配置模板（含手动模板和定时任务模板）。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 主键，自增 |
| project_id | int | 项目ID |
| task_type | int | 任务类型：1=App，3=Web，5=PC，1000=用例驱动 |
| suite_id | int | 关联应用ID（App类型时使用） |
| task_name | varchar | 模板名称 |
| task_desc | varchar | 模板描述（用例模板用） |
| task_template_status | int | 模板状态（TaskTemplateStatusEnum） |
| task_template_check_status | int | 模板检查状态 |
| template_type | int | 模板类型：1=普通模板，2=定时任务模板 |
| cron_expression | varchar | Cron 表达式（定时任务时使用） |
| cron_rule | varchar | 老版本定时高级配置 |
| create_user_id | int | 创建人 |
| update_user_id | int | 更新人 |
| create_time | timestamp | 创建时间 |
| update_time | timestamp | 更新时间 |
| status | int | 数据状态：0=删除，1=有效 |
| contains_app_script | tinyint | 是否含App脚本 |
| contains_web_script | tinyint | 是否含Web脚本 |
| contains_pc_script | tinyint | 是否含PC脚本 |

- 关联接口：
  - [TaskTemplateController](../../任务管理服务/07-开放接口文档/任务模板管理/TaskTemplateController.md)

- 关联子表：
  - `task_template_detail` — 执行详细配置
  - `task_template_script` — 脚本关联
  - `task_template_device` — 设备关联
  - `task_template_case` — 用例关联
  - `task_template_notice` — 通知关联
