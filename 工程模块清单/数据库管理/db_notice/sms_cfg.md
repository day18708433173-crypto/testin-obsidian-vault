# sms_cfg (db_notice)

- 用途：短信配置，存储短信服务平台的 API 地址、密钥等。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| sms_name | varchar(255) | 配置名称 |
| api_url | varchar(255) | API 地址 |
| api_key | varchar(255) | API Key |
| api_secret | varchar(255) | API Secret |
| status | int(11) | 状态 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[service-SmsCfg](../../平台基础功能服务/07-开放接口文档/通知与消息/service-SmsCfg.md)
