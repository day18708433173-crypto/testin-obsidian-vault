# source_sql (db_source)

- 用途：SQL 表达式管理。存储数据源中配置的 SQL 查询语句。
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | bigint(20) | 主键 |
| source_config_id | bigint(20) | 所属数据源 ID |
| name | varchar(60) | SQL 标题 |
| content | text | SQL 表达式 |
| db_name | varchar(255) | 数据库实例名字 |
| env_id | int(11) | 环境 ID |
| db_alias | varchar(255) | 数据库别名 |
| db_config_id | int(11) | 数据源管理里的数据库配置 ID |
| status | tinyint(4) | 0=逻辑删除 1=正常 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 索引：`source_config_id`、`updatetime`

- 关联接口：
  - [service-SqlCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-SqlCtrl.md)
