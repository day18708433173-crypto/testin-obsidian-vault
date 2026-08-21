# normal_value_info (db_source)

- 用途：普通数据源单元格值，按（行号, 列号）坐标存储每个单元格的内容。
- 实体类：`cn.testin.pojo.normal.NormalValueInfo`
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | bigint(20) | 主键（自增） |
| eid | int(11) | 企业 ID |
| project_id | int(11) | 项目组 ID |
| normal_source_id | bigint(20) | 所属实例表 ID（关联 [normal_source](normal_source.md)） |
| param_value | varchar | 单元格值 |
| col_index | int(11) | 列号 |
| row_index | int(11) | 行号 |
| update_user | varchar(30) | 更新人 |
| create_time | bigint(20) | 创建时间 |
| update_time | bigint(20) | 更新时间 |

- 与 [case_value_info](case_value_info.md)（用例表族）定位相同，是普通表族的单元格值表。

- 关联接口：
  - [NormalSourceController](../../数据源/07-开放接口文档/数据源管理/NormalSourceController.md)
