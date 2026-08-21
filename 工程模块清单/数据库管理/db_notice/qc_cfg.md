# qc_cfg (db_notice)

- 用途：企业微信（企微）配置，存储企微机器人的 Webhook 地址等。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| qc_name | varchar(255) | 配置名称 |
| webhook_url | varchar(1024) | Webhook 地址 |
| status | int(11) | 状态 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[service-QcCfg](../../平台基础功能服务/07-开放接口文档/其他ApiServlet/service-QcCfg.md)
