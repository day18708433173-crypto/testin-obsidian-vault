# sms_templet (db_notice)

- 用途：短信模板，定义短信发送内容模板。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| templet_code | varchar(255) | 模板编码 |
| templet_name | varchar(255) | 模板名称 |
| content | text | 模板内容 |
| status | int(11) | 状态 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 关联接口：[service-SmsTemplet](../../平台基础功能服务/07-开放接口文档/通知与消息/service-SmsTemplet.md)
