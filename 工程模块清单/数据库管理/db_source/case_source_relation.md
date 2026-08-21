# case_source_relation (db_source)

- 用途：用例与数据源绑定关系。将测试用例与用例数据源实例表关联，并可指定具体行用于回放调试。
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| case_id | int(11) | 用例 ID（主键） |
| case_source_id | int(11) | 用例实例表 ID（关联 case_source.id） |
| case_table_row_id | int(11) | 用例实例表某一行 ID，用于回放中调试指定行 |

- 唯一约束：`idx_case_id` (case_id) — 一个用例只能绑定一个数据源

- 关联接口：
  - [CaseSourceController](../../数据源/07-开放接口文档/数据源管理/CaseSourceController.md)（add_case_relation / remove_case_relation / sync_case / edit_case_row_id / unbind_case_row_id）
