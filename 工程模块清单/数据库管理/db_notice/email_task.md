# email_task (db_notice)

- 用途：邮件发送任务，记录待发送邮件的收件人、标题、内容等信息。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| email_cfg_id | int(11) | 邮件配置 ID |
| to_users | text | 收件人列表 |
| title | varchar(255) | 邮件标题 |
| content | text | 邮件内容 |
| send_status | int(11) | 发送状态 |
| create_time | bigint(20) | 创建时间 |
| update_time | bigint(20) | 更新时间 |

- 关联接口：[service-EmailTask](../../平台基础功能服务/07-开放接口文档/通知与消息/service-EmailTask.md)
