# normal_source (db_source)

- 用途：普通数据源树（目录/实例表），通用表格数据的存储组织。
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | bigint(20) | 主键 |
| eid | int(11) | 企业 ID |
| project_id | int(11) | 项目组 ID |
| parent_id | bigint(20) | 父节点 ID（自关联树） |
| name | varchar(255) | 名称 |
| type | tinyint(4) | 类型：0=目录 1=实例表 |
| status | tinyint(4) | 0=逻辑删除 1=正常 |
| create_user | varchar(30) | 创建人 |
| update_user | varchar(30) | 更新人 |
| create_time | bigint(20) | 创建时间 |
| update_time | bigint(20) | 更新时间 |

- 索引：`parent_id`、`eid_project_id`

- 关联接口：
  - [NormalSourceController](../../数据源/07-开放接口文档/数据源管理/NormalSourceController.md)
