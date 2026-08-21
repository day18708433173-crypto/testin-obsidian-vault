# system_param (db_user)

- 用途：系统参数配置，存储 OSS、短信、邮件等各服务参数。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| param_name | varchar(255) | 参数名 |
| param_value | varchar(1024) | 参数值 |
| status | int(11) | 状态 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[service-SystemParam](../../平台基础功能服务/07-开放接口文档/用户与认证/service-SystemParam.md)
