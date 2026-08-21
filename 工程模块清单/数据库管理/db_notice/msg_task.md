# msg_task (db_notice)

- 用途：消息任务（通用），支持站内信等多种消息类型。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| msg_type | int(11) | 消息类型 |
| receive_user_id | int(11) | 接收人 |
| title | varchar(255) | 标题 |
| content | text | 内容 |
| send_status | int(11) | 发送状态 |
| create_time | bigint(20) | 创建时间 |
| update_time | bigint(20) | 更新时间 |

- 关联接口：[service-MsgTask](../../平台基础功能服务/07-开放接口文档/通知与消息/service-MsgTask.md)
