# sms_task (db_notice)

- 用途：短信发送任务，记录待发送短信的目标手机号、内容等。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| sms_cfg_id | int(11) | 短信配置 ID |
| phone_numbers | text | 目标手机号 |
| content | text | 短信内容 |
| send_status | int(11) | 发送状态 |
| create_time | bigint(20) | 创建时间 |
| update_time | bigint(20) | 更新时间 |

- 关联接口：[service-SmsTask](../../平台基础功能服务/07-开放接口文档/通知与消息/service-SmsTask.md)
