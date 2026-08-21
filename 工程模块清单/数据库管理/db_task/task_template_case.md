# task_template_case (db_task)

- 用途：模板-用例关联
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| task_template_id | int | 关联的任务模板id |
| case_id | int | 用例Id |
| case_name | varchar | 用例名称 |
| order_num | int | 用例顺序，组织报告使用？？ |
| case_check_status | int | 用例检查状态 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 关联接口：
  - [TaskTemplateController](../../任务管理服务/07-开放接口文档/任务模板管理/TaskTemplateController.md)

- 关联表：
  - 主表 [task_template](task_template.md)
