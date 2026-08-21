# admin_user_role (db_admin)

- 用途：后台用户-角色关联，管理后台管理员的角色分配。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| uid | int(11) | 后台用户 ID |
| role_id | int(11) | 角色 ID |
| createtime | bigint(20) | 创建时间 |

- 关联接口：[service-Admin](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Admin.md)
