# datatable_values (db_source)

- 用途：数据表单元格值。每个单元格由 (source_config_id, row_index, col_index) 唯一确定。
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | bigint(20) | 主键 |
| eid | int(11) | 企业 ID |
| projectid | int(11) | 项目组 ID |
| source_config_id | bigint(20) | 关联的数据表 ID |
| param_value | text | 变量值（文本内容） |
| param_type | int(11) | 备用字段：1=字符串 2=数字 99=引用类型 |
| value_type | int(11) | 值来源类型：0=手动输入 1=全局表引用 2=SQL生成 3=随机生成 |
| row_index | int(11) | 行号（从 0 开始） |
| col_index | int(11) | 列号（从 0 开始） |
| sync_status | int(11) | 同步状态（默认 0） |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |
| update_by | varchar(255) | 修改人 |

- 唯一约束：`unique_cell` (source_config_id, row_index, col_index) — 一个单元格只能有一个值
- 索引：`eid_projectid`、`updatetime`

- 关联接口：
  - [service-DataTableCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-DataTableCtrl.md)
  - [service-SelectCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-SelectCtrl.md)
