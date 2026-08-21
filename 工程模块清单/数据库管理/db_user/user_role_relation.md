# user_role_relation (db_user)

- 用途：用户与角色的关联关系，记录用户在特定项目下的角色。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| userid | int(11) | 用户 ID |
| roleid | int(11) | 角色 ID |
| eid | int(11) | 企业 ID |
| projectid | int(11) | 项目 ID |
| createtime | bigint(20) | 创建时间 |

- 关联接口：[UserController](../../平台基础功能服务/07-开放接口文档/用户与认证/UserController.md)、[service-Role](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Role.md)
