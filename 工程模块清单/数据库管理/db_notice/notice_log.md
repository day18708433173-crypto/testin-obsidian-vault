# notice_log (db_notice)

- 用途：通知发送日志，记录每次通知的发送结果。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| event_id | int(11) | 事件 ID |
| channel_cfg_id | int(11) | 渠道配置 ID |
| receive_user_id | int(11) | 接收人 ID |
| send_status | int(11) | 发送状态 |
| send_result | text | 发送结果 |
| create_time | datetime | 创建时间 |

- 关联接口：[NoticeLogController](../../平台基础功能服务/07-开放接口文档/通知与消息/NoticeLogController.md)
