# notice_channel_cfg (db_notice)

- 用途：通知渠道配置，关联事件与具体发送渠道（邮箱/短信/企微）。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| channel_name | varchar(128) | 渠道名称 |
| channel_type | int(11) | 渠道类型 |
| status | int(11) | 状态 |
| create_time | datetime | 创建时间 |

- 关联接口：[NoticeChannelCfgController](../../平台基础功能服务/07-开放接口文档/通知与消息/NoticeChannelCfgController.md)
