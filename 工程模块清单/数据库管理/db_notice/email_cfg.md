# email_cfg (db_notice)

- 用途：邮件服务器配置，存储 SMTP 服务器地址、端口、账号密码等。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| email | varchar(128) | 发件邮箱账号 |
| pwd | varchar(128) | 邮箱密码 |
| nick_name | varchar(128) | 发件昵称 |
| host | varchar(128) | SMTP 服务器地址 |
| port | int(11) | 端口 |
| ssl | smallint(1) | 是否 SSL |
| type | int(11) | 邮箱类型 |
| create_time | bigint(20) | 创建时间 |
| update_time | bigint(20) | 更新时间 |

- 关联接口：[service-EmailCfg](../../平台基础功能服务/07-开放接口文档/通知与消息/service-EmailCfg.md)
