# system_config (db_user)

- 用途：系统级别配置，如邮件服务器、文件存储路径等全局设置。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| config_key | varchar(255) | 配置键 |
| config_value | varchar(1024) | 配置值 |
| descr | varchar(255) | 描述 |
| status | int(11) | 状态 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[service-SystemCfg](../../平台基础功能服务/07-开放接口文档/用户与认证/service-SystemCfg.md)
