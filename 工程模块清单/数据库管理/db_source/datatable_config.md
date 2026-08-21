# datatable_config (db_source)

- 用途：数据表行列配置，存储行的顺序、列的顺序、行列最大值，用于渲染数据表格。
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | int(11) | 主键 |
| eid | int(11) | 企业 ID |
| projectid | int(11) | 项目组 ID |
| source_config_id | bigint(20) | 关联的数据表 ID（对应 source_config.id，type=2/3） |
| row_order | text | 行的排序（JSON 数组格式） |
| col_order | text | 列的排序（JSON 数组格式） |
| row_max_index | int(10) | 表中行最大值（下一个插入行的序号） |
| col_max_index | int(10) | 表中列最大值（下一个插入列的序号） |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 索引：`eid_projectid`、`source_config_id`、`updatetime`

- 关联接口：
  - [service-DataTableCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-DataTableCtrl.md)
