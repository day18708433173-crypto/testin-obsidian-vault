# notice_event (db_notice)

- 用途：通知事件定义，定义系统中各类触发通知的事件。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| event_code | varchar(255) | 事件编码 |
| event_name | varchar(255) | 事件名称 |
| event_type | int(11) | 事件类型 |
| status | int(11) | 状态 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 关联接口：[NoticeEventController](../../平台基础功能服务/07-开放接口文档/通知与消息/NoticeEventController.md)
