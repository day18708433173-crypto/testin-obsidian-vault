# message_task (db_notice)

- 用途：消息任务（旧版通用），兼容多种消息类型的发送。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| task_type | int(11) | 任务类型 |
| target | varchar(255) | 目标（手机/邮箱/URL） |
| content | text | 内容 |
| status | int(11) | 状态 |
| create_time | bigint(20) | 创建时间 |
| update_time | bigint(20) | 更新时间 |

- 关联接口：[service-MsgTask](../../平台基础功能服务/07-开放接口文档/通知与消息/service-MsgTask.md)
