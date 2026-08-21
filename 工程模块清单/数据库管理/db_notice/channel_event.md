# channel_event (db_notice)

- 用途：渠道与事件的关联表，定义某渠道订阅了哪些事件。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| channel_id | int(11) | 渠道 ID |
| event_id | int(11) | 事件 ID |
| status | int(11) | 状态 |
| create_time | datetime | 创建时间 |

- 关联接口：[NoticeEventController](../../平台基础功能服务/07-开放接口文档/通知与消息/NoticeEventController.md)
