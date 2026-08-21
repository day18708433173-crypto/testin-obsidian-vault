# script_statistic (db_plan)

- 用途：脚本统计数据。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| project_id | int(11) | 项目 ID |
| script_no | int(11) | 脚本编号 |
| stat_type | int(11) | 统计类型 |
| stat_value | varchar(255) | 统计值 |
| create_time | timestamp | 创建时间 |

- 关联接口：[ScriptStatisticController](../../平台基础功能服务/07-开放接口文档/测试计划/ScriptStatisticController.md)
