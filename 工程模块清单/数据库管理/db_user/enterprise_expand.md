# enterprise_expand (db_user)

- 用途：企业扩展信息，存储企业自定义字段等附加数据。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| eid | int(11) | 企业 ID |
| ext_key | varchar(255) | 扩展键 |
| ext_value | text | 扩展值 |

- 关联接口：[service-Enterprise](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Enterprise.md)
