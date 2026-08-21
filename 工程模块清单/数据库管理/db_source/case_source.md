# case_source (db_source)

- 用途：用例数据源树。按目录/实例表/数据源三级组织，用例执行时的数据来源。
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | bigint(20) | 主键 |
| eid | int(11) | 企业 ID |
| project_id | int(11) | 项目组 ID |
| parent_id | bigint(20) | 父节点 ID（自关联树） |
| name | varchar(255) | 名称（utf8 编码） |
| case_source_order | int(11) | 排序（目录和实例表分开单独排序） |
| type | tinyint(4) | 类型：0=目录 1=实例表 2=数据源 |
| status | tinyint(4) | 0=逻辑删除 1=正常 |
| create_user_id | int(11) | 创建人 ID |
| update_user_id | int(11) | 更新人 ID |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 索引：`parent_id`、`eid_project_id`

- 关联接口：
  - [CaseSourceController](../../数据源/07-开放接口文档/数据源管理/CaseSourceController.md)
