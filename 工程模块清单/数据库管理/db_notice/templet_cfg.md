# templet_cfg (db_notice)

- 用途：模板配置管理，管理各类通知模板的基础信息。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| templet_type | int(11) | 模板类型 |
| templet_name | varchar(255) | 模板名称 |
| status | int(11) | 状态 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[service-EmailTempletCfg](../../平台基础功能服务/07-开放接口文档/通知与消息/service-EmailTempletCfg.md)
