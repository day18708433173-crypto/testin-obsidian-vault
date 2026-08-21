# task_template_notice (db_task)

- 用途：模板-通知配置关联
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 主键 |
| task_template_id | int | 关联的任务模板id |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 关联接口：
  - [TaskTemplateController](../../任务管理服务/07-开放接口文档/任务模板管理/TaskTemplateController.md)

- 关联表：
  - 主表 [task_template](task_template.md)
