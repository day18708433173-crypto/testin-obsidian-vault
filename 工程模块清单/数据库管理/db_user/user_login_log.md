# user_login_log (db_user)

- 用途：用户登录日志，记录每次登录的 IP、时间、登录类型等。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| userid | int(11) | 用户 ID |
| login_ip | varchar(128) | 登录 IP 地址 |
| login_time | bigint(20) | 登录时间 |
| login_type | int(11) | 登录类型 |
| status | int(11) | 状态 |

- 关联接口：[service-Login](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Login.md)
