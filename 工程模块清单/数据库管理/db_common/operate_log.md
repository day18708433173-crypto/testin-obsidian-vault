# operate_log (db_common)

- 用途：后台操作日志，记录管理员的后台操作。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| operate_type | int(11) | 操作类型 |
| operate_user_id | int(11) | 操作人 ID |
| operate_content | text | 操作内容 |
| operate_result | varchar(255) | 操作结果 |
| create_time | datetime | 创建时间 |

- 关联接口：[OperateLogController](../../平台基础功能服务/07-开放接口文档/基础设施与统计/OperateLogController.md)
