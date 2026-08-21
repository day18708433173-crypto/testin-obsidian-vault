# plan_email_notice (db_plan)

- 用途：测试计划邮件通知配置，设置完成/取消失败后的邮件接收人。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| plan_info_id | bigint(20) | 测试计划 ID |
| email_status | int(11) | 邮件开关（0=关 1=开） |
| finish_notice_leader | text | 完成通知接收人列表 |
| cancel_notice_leader | text | 取消通知接收人列表 |
| create_user_id | int(11) | 创建人 |
| update_user_id | int(11) | 更新人 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[EmailNoticeController](../../平台基础功能服务/07-开放接口文档/测试计划/EmailNoticeController.md)
