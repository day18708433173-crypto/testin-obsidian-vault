# app_user_relation (db_user)

- 用途：第三方应用（微信/钉钉等）与 Testin 用户绑定关系。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(11) | 主键 |
| app_name | varchar(64) | 第三方应用名称 |
| openid | varchar(128) | 用户 OpenID |
| unionid | varchar(128) | 微信 UnionID |
| userid | int(11) | Testin 用户 ID |
| status | int(11) | 绑定状态（1=绑定 0=解绑） |
| bindtime | bigint(20) | 绑定时间 |

- 关联接口：[service-Login](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Login.md)
