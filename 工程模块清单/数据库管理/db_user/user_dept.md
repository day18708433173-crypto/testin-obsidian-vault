# user_dept (db_user)

- 用途：用户部门信息。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| dept_name | varchar(255) | 部门名称 |
| eid | int(11) | 企业 ID |
| status | int(11) | 状态 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[UserController](../../平台基础功能服务/07-开放接口文档/用户与认证/UserController.md)
