# audit_info (db_user)

- 用途：企业认证审计数据，存储审核历史记录。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| eid | int(11) | 企业 ID |
| audit_type | int(11) | 审计类型 |
| audit_status | int(11) | 审核状态 |
| audit_content | text | 审核内容 |
| operator_userid | int(11) | 操作人 ID |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[service-AuditData](../../平台基础功能服务/07-开放接口文档/其他ApiServlet/service-AuditData.md)
