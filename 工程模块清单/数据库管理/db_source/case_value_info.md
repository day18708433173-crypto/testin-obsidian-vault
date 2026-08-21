# case_value_info (db_source)

- 用途：用例数据源单元格值。
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | bigint(20) | 主键 |
| eid | int(11) | 企业 ID |
| project_id | int(11) | 项目组 ID |
| case_source_id | bigint(20) | 关联的用例数据表 ID |
| param_value | text | 变量值 |
| row_index | int(11) | 行号 |
| col_index | int(11) | 列号 |
| update_user_id | int(11) | 更新人 ID |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 唯一约束：`unique_cell` (case_source_id, row_index, col_index)
- 索引：`eid_projectid`、`updatetime`

- 关联接口：
  - [CaseSourceController](../../数据源/07-开放接口文档/数据源管理/CaseSourceController.md)
