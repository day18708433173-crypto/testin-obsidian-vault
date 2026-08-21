# notice_info (db_task)

- 用途：通知信息通用表（嵌入各通知配置）
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| notice_type | int | 发送通知类型，1为任务完成发送各种通知。2任务完成发送邮件，3. 脚本失败发送通知 |
| notice_status | int | 通知状态，1为开启，0为关闭 |
| notice_condition | varchar | 通知发送的条件。json对象 |
| notice_object | varchar | 通知的对象 |

- 关联接口：
  - [TaskTemplateController](../../任务管理服务/07-开放接口文档/任务模板管理/TaskTemplateController.md)
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)

> 说明：`NoticeInfo` 为嵌入式通用对象（无独立 `@TableName` 实体注解），其字段以列形式嵌入各通知配置记录中。
