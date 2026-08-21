# normal_column_info (db_source)

- 用途：普通数据源列定义，存储实例表的列（变量）信息。
- 实体类：`cn.testin.pojo.normal.NormalColumnInfo`
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | int(11) | 主键（自增） |
| eid | int(11) | 企业 ID |
| project_id | int(11) | 项目组 ID |
| normal_source_id | bigint(20) | 所属实例表 ID（关联 [normal_source](normal_source.md)） |
| name | varchar | 列名 |
| desc | varchar | 列描述 |
| type | tinyint(4) | 列类型，默认字符串 |
| col_index | int(11) | 列顺序 |
| update_user | varchar(30) | 更新人 |
| create_time | bigint(20) | 创建时间 |
| update_time | bigint(20) | 更新时间 |

- 与 [case_column_info](case_column_info.md)（用例表族）字段结构基本一致，是普通表族的列定义表。

- 关联接口：
  - [NormalSourceController](../../数据源/07-开放接口文档/数据源管理/NormalSourceController.md)
