# execute_record_config (db_plan)

- 用途：执行记录配置，存储执行时的额外配置参数。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| execute_record_id | bigint(20) | 执行记录 ID |
| config_key | varchar(255) | 配置键 |
| config_value | text | 配置值 |
| create_time | timestamp | 创建时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[ExecuteRecordController](../../平台基础功能服务/07-开放接口文档/测试计划/ExecuteRecordController.md)
