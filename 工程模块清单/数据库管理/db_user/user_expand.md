# user_expand (db_user)

- 用途：用户扩展信息，存储用户自定义扩展字段。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| userid | int(11) | 用户 ID |
| ext_key | varchar(255) | 扩展键 |
| ext_value | text | 扩展值 |
| createtime | bigint(20) | 创建时间 |

- 关联接口：[service-User](../../平台基础功能服务/07-开放接口文档/用户与认证/service-User.md)
