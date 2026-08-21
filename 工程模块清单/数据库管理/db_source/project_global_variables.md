# project_global_variables (db_source)

- 用途：项目全局变量。支持普通变量和加密变量（RSA 加密存储）。
- 主要字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | int(11) | 主键 |
| project_id | int(8) | 项目 ID |
| type | tinyint(2) | 类型：1=普通变量 2=加密变量 |
| variable_name | varchar(255) | 变量名 |
| variable_value | text | 变量值（加密变量存储加密后的值） |
| variable_type | tinyint(2) | 变量数据类型：0=默认 1=String 2=数字 3=对象 4=数组 |
| variable_desc | varchar(255) | 变量描述 |
| status | tinyint(1) | 0=逻辑删除 1=正常 |
| createtime | bigint(20) | 创建时间 |
| updatetime | bigint(20) | 更新时间 |

- 唯一约束：`project_var_name_index` (project_id, variable_name) — 每个项目下变量名唯一

- 关联接口：
  - [ProjectGlobalVariablesController](../../数据源/07-开放接口文档/数据源管理/ProjectGlobalVariablesController.md)
