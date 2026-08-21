# enterprise_info (db_user)

- 用途：企业基本信息，存储注册企业的 ID、名称、状态、认证信息等。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 主键，企业 ID |
| e_short_name | varchar(128) | 企业短名称 |
| e_display_name | varchar(128) | 企业显示名称 |
| e_trade_name | varchar(128) | 行业 |
| e_status | int(11) | 企业状态 |
| e_login_domain | varchar(255) | 企业登录域名 |
| max_user_num | int(11) | 最大用户数 |
| max_project_num | int(11) | 最大项目数 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[EnterpriseController](../../平台基础功能服务/07-开放接口文档/用户与认证/EnterpriseController.md)、[service-Enterprise](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Enterprise.md)
