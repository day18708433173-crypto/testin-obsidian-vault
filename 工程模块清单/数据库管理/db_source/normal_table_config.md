# normal_table_config (db_source)

- 用途：普通数据源表行列配置，记录行/列的排列顺序与当前最大行列号。
- 实体类：`cn.testin.pojo.normal.NormalTableConfig`
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | int(11) | 主键（自增） |
| eid | int(11) | 企业 ID |
| project_id | int(11) | 项目组 ID |
| normal_source_id | bigint(20) | 所属实例表 ID（关联 [normal_source](normal_source.md)） |
| row_order | varchar | 行顺序（行号序列） |
| col_order | varchar | 列顺序（列号序列） |
| row_max_index | int(11) | 行最大值 |
| col_max_index | int(11) | 列最大值 |
| create_time | bigint(20) | 创建时间 |
| update_time | bigint(20) | 更新时间 |

- 与 [case_table_config](case_table_config.md)（用例表族）、[datatable_config](datatable_config.md)（legacy 表族）职责相同。

- 关联接口：
  - [NormalSourceController](../../数据源/07-开放接口文档/数据源管理/NormalSourceController.md)
