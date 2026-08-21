# case_column_info (db_source)

- 用途：用例数据源列定义。每行定义一个列（变量）。
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | int(11) | 主键 |
| eid | int(11) | 企业 ID |
| project_id | int(11) | 项目组 ID |
| case_source_id | bigint(20) | 关联的用例数据表 ID |
| name | varchar(190) | 列名（变量名），utf8mb4_bin 排序 |
| type | tinyint(2) | 列类型：1=字符串 |
| desc | varchar(255) | 列的备注 |
| col_index | int(11) | 列号 |
| show_in_report | int(2) | 是否在报告中展示：0=不展示 1=展示 |
| update_user_id | int(11) | 更新人 ID |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 唯一约束：`unique_colIndex` (case_source_id, col_index)、`unique_name` (case_source_id, name)
- 索引：`eid_projectid`、`updatetime`

- 关联接口：
  - [CaseSourceController](../../数据源/07-开放接口文档/数据源管理/CaseSourceController.md)
