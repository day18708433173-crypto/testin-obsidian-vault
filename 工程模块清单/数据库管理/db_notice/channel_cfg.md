# channel_cfg (db_notice)

- 用途：通知渠道基础配置（旧版），记录渠道类型和状态。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| channel_name | varchar(128) | 渠道名称 |
| type | int(11) | 渠道类型（0=短信 1=企微） |
| descr | varchar(255) | 描述 |
| status | int(11) | 状态 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[service-Channel](../../平台基础功能服务/07-开放接口文档/通知与消息/service-Channel.md)
