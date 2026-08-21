# enterprise_channel_relation (db_notice)

- 用途：企业-渠道关联，配置企业可使用哪些通知渠道。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| eid | int(11) | 企业 ID |
| channel_type | int(11) | 渠道类型 |
| status | int(11) | 状态 |
| create_time | datetime | 创建时间 |

- 关联接口：[service-ChannelCfg](../../平台基础功能服务/07-开放接口文档/通知与消息/service-ChannelCfg.md)
