# user_region (db_user)

- 用途：用户区域信息，记录用户所属地理区域。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| region_name | varchar(255) | 区域名称 |
| eid | int(11) | 企业 ID |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[service-User](../../平台基础功能服务/07-开放接口文档/用户与认证/service-User.md)
