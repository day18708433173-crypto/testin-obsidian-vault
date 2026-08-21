# user_info (db_user)

- 用途：用户基本信息，存储账号、邮箱、手机号、状态等。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键，用户 ID |
| user_name | varchar(255) | 用户账号名 |
| password | varchar(255) | 密码（加密） |
| nick_name | varchar(255) | 昵称 |
| email | varchar(128) | 邮箱 |
| phone | varchar(32) | 手机号 |
| status | int(11) | 状态 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[UserController](../../平台基础功能服务/07-开放接口文档/用户与认证/UserController.md)、[service-Login](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Login.md)、[service-User](../../平台基础功能服务/07-开放接口文档/用户与认证/service-User.md)
