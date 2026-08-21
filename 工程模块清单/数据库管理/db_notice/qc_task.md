# qc_task (db_notice)

- 用途：企业微信消息发送任务。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| qc_cfg_id | int(11) | 企微配置 ID |
| content | text | 消息内容 |
| send_status | int(11) | 发送状态 |
| create_time | bigint(20) | 创建时间 |
| update_time | bigint(20) | 更新时间 |

- 关联接口：[service-QcTask](../../平台基础功能服务/07-开放接口文档/其他ApiServlet/service-QcTask.md)
