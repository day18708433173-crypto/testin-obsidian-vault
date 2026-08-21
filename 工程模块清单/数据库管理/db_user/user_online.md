# user_online (db_user)

- 用途：用户在线状态记录，记录用户当前所在项目和活跃时间。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| userid | int(11) | 用户 ID |
| projectid | int(11) | 当前所在项目 ID |
| logindate | bigint(20) | 登录时间 |
| last_update_date | bigint(20) | 最后活跃时间 |
| login_type | int(11) | 登录类型 |

- 关联接口：[service-Online](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Online.md)
