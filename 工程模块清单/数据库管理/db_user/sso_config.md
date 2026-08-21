# sso_config (db_user)

- 用途：SSO 单点登录配置，支持 CAS、OAuth、钉钉、企业微信等登录方式。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| eid | int(11) | 企业 ID |
| sso_type | int(11) | SSO 类型 |
| sso_config | text | SSO 配置 JSON |
| status | int(11) | 状态 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[service-Login](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Login.md)
