# project_info (db_user)

- 用途：项目基本信息，存储项目名称、所属企业、状态等。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键，项目 ID |
| eid | int(11) | 所属企业 ID |
| project_name | varchar(255) | 项目名称 |
| project_desc | varchar(255) | 项目描述 |
| status | int(11) | 状态 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[ProjectController](../../平台基础功能服务/07-开放接口文档/用户与认证/ProjectController.md)、[service-Project](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Project.md)
