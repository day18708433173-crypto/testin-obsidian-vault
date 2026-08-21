# email_templet_config (db_notice)

- 用途：邮件模板配置，管理邮件模板的启用和数据。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| templet_code | varchar(255) | 模板编码 |
| templet_name | varchar(255) | 模板名称 |
| title | varchar(255) | 标题 |
| content | text | 内容 |
| status | int(11) | 状态 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 关联接口：[EmailTemplateCfgController](../../平台基础功能服务/07-开放接口文档/通知与消息/EmailTemplateCfgController.md)
