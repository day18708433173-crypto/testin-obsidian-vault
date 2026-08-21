# admin_role (db_admin)

- 用途：后台角色定义，管理后台角色及权限。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| role_name | varchar(255) | 角色名称 |
| descr | varchar(255) | 角色描述 |
| status | int(11) | 状态 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[service-Admin](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Admin.md)
