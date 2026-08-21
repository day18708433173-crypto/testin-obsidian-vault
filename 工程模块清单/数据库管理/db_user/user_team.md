# user_team (db_user)

- 用途：用户团队信息，支持团队协作。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| team_name | varchar(255) | 团队名称 |
| eid | int(11) | 企业 ID |
| projectid | int(11) | 项目 ID |
| createtime | bigint(20) | 创建时间 |

- 关联接口：[ProjectController](../../平台基础功能服务/07-开放接口文档/用户与认证/ProjectController.md)
