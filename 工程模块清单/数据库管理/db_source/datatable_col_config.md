# datatable_col_config (db_source)

- 用途：数据表列定义。每行定义一个列（变量），包含变量名、类型、作用域、报告可见性等属性。
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | int(11) | 主键 |
| eid | int(11) | 企业 ID |
| project_id | int(11) | 项目组 ID |
| source_config_id | bigint(100) | 关联的数据表 ID |
| name | varchar(190) | 列名（变量名），utf8mb4_bin 排序 |
| type | int(11) | 列类型：0=默认 1=String 2=数字 3=对象 4=数组 |
| scope | varchar(10) | 变量类型：GLOBAL=全局变量 LOCAL=局部变量（z7720+） |
| descr | varchar(255) | 列的备注 |
| tmp | int(11) | 临时变量标记（已废弃，z7720 去除） |
| quote_type | int(11) | 数值引用类型：1=全局整列引用 2=手动设置单元格数据 |
| col_index | int(11) | 列号（从 0 开始） |
| show_in_report | tinyint(1) | 是否在报告执行摘要中显示：1=显示 0=不显示 |
| sql_col | varchar(255) | 关联的 SQL 查询结果列名 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |
| update_by | varchar(255) | 修改人 |

- 唯一约束：`unique_colIndex` (source_config_id, col_index) — 每个表只能有一个同列号
- 唯一约束：`unique_name` (source_config_id, name) — 每个表只能有一个同名变量
- 索引：`eid_projectid`、`type`、`updatetime`、`source_id_show_in_report`

- 关联接口：
  - [service-ColConfigCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-ColConfigCtrl.md)
  - [service-DataTableCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-DataTableCtrl.md)
