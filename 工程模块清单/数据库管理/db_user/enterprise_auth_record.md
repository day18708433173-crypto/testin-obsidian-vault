# enterprise_auth_record (db_user)

- 用途：企业认证审计记录，存储企业提交的认证信息及审核状态。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| eid | int(11) | 企业 ID |
| auth_name | varchar(128) | 认证名称 |
| auth_status | int(11) | 认证状态 |
| auth_content | text | 认证内容 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[EnterpriseController](../../平台基础功能服务/07-开放接口文档/用户与认证/EnterpriseController.md)、[service-AuditData](../../平台基础功能服务/07-开放接口文档/其他ApiServlet/service-AuditData.md)
