# email_templet (db_notice)

- 用途：邮件模板（旧版），定义邮件标题和内容模板。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| templet_name | varchar(255) | 模板名称 |
| title | varchar(255) | 标题模板 |
| content | text | 内容模板 |
| status | int(11) | 状态 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[service-EmailTempletCfg](../../平台基础功能服务/07-开放接口文档/通知与消息/service-EmailTempletCfg.md)
