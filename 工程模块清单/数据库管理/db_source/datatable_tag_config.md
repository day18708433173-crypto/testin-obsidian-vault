# datatable_tag_config (db_source)

- 用途：legacy 数据表行标签关联。将标签绑定到数据表的特定行。
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | bigint(20) | 主键 |
| source_config_id | bigint(20) | 数据表 ID |
| row_index | int(11) | 行号 |
| tag_id | bigint(20) | 标签 ID（关联 tag_info.id） |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |
| update_by | varchar(255) | 修改人 |

- 唯一约束：`unique_tag` (source_config_id, row_index, tag_id) — 同一行不能有重复的标签
- 索引：`updatetime`

- 关联接口：
  - [service-DataTableCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-DataTableCtrl.md)（configTag / deleteTag）
