# task_template_case_status_sync (db_task)

- 用途：模板用例ID状态同步
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint |  |
| case_id | int |  |
| case_check_status | int |  |
| sync_count | int |  |

- 关联接口：
  - [TaskTemplateController](../../任务管理服务/07-开放接口文档/任务模板管理/TaskTemplateController.md)

- 关联表：
  - 主表 [task_template](task_template.md)
