# source_config (db_source)

- 用途：数据源配置主表，树形结构组织的核心表。通过 `parent_id` 自关联实现目录树。
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | bigint(20) | 主键 |
| eid | bigint(20) | 企业 ID |
| project_id | int(11) | 项目组 ID |
| source_id | bigint(20) | 所属数据源 ID（根节点标识） |
| parent_id | bigint(20) | 父节点 ID（自关联树） |
| name | varchar(255) | 名称 |
| type | tinyint(4) | 类型：0=目录 1=数据源(环境数据源20) 2=全局表 3=实例表 4=SQL语句管理 5=具体SQL |
| sql_id | bigint(20) | 关联的 SQL ID（type=5 时） |
| bind | int(11) | 是否绑定默认脚本 1=是 0=否 |
| script_no | int(11) | 脚本编号 |
| script_type | int(11) | 脚本类型：1=app 3=web 5=pc |
| env_id | int(11) | 环境 ID |
| db_alias | varchar(255) | 数据库别名 |
| db_config_id | int(11) | 数据源管理里的数据库配置 ID |
| current_order | int(4) | 在当前目录下的排序位置 |
| update_by | varchar(255) | 修改人 |
| status | tinyint(4) | 0=逻辑删除 1=正常（MyBatis Plus 逻辑删除字段） |
| createtime | bigint(20) | 创建时间（Unix 时间戳） |
| updatetime | bigint(20) | 更新时间（Unix 时间戳） |

- 唯一约束：`unique_scriptno` (source_id, script_no, status) — 同一数据源下脚本编号唯一
- 索引：`eid_projectid`、`parent_id`、`type`、`script_no`、`updatetime`、`source_id`

- 关联接口：
  - [DataSourceController](../../数据源/07-开放接口文档/数据源管理/DataSourceController.md)
  - [service-SourceConfigCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-SourceConfigCtrl.md)
  - [service-SelectCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-SelectCtrl.md)
