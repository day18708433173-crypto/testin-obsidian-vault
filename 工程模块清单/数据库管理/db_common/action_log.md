# action_log (db_common)

- 用途：用户操作行为日志，记录用户在系统中的关键操作。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| user_id | int(11) | 用户 ID |
| user_name | varchar(255) | 用户名 |
| eid | int(11) | 企业 ID |
| action_type | varchar(255) | 操作类型 |
| action_content | text | 操作内容 |
| create_time | datetime | 创建时间 |

- 关联接口：[service-ActionLogController](../../平台基础功能服务/07-开放接口文档/其他ApiServlet/service-ActionLogController.md)
