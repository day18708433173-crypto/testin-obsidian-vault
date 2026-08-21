# enterprise_user_relation (db_user)

- 用途：企业用户与项目的关联关系，记录用户在项目中的角色。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| eid | int(11) | 企业 ID |
| userid | int(11) | 用户 ID |
| projectid | int(11) | 项目 ID |
| roleid | int(11) | 角色 ID |
| status | int(11) | 状态 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[EnterpriseController](../../平台基础功能服务/07-开放接口文档/用户与认证/EnterpriseController.md)、[service-Project](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Project.md)
