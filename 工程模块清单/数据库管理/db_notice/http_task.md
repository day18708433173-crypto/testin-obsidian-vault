# http_task (db_notice)

- 用途：HTTP 回调任务，向指定 URL 发送通知回调。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| url | varchar(1024) | 回调 URL |
| method | varchar(10) | 请求方法 |
| params | text | 请求参数 |
| headers | text | 请求头 |
| send_status | int(11) | 发送状态 |
| retry_count | int(11) | 重试次数 |
| create_time | bigint(20) | 创建时间 |
| update_time | bigint(20) | 更新时间 |

- 关联接口：[service-HttpTask](../../平台基础功能服务/07-开放接口文档/通知与消息/service-HttpTask.md)
