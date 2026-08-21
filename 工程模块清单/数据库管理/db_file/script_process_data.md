# script_process_data

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 回收与恢复 |
| 关联 Mapper | ScriptProcessDataMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | — | PK | 主键ID |
| script_no | — | | 脚本编号 |
| project_id | — | | 项目ID |
| script_type | — | | 脚本类型 |
| data | — | | 数据 |
| create_user_id | — | | 创建人ID |
| update_user_id | — | | 更新人ID |
| create_time | Long | | 创建时间 |
| update_time | Long | | 更新时间 |

## 关联关系

- 引用：[script_file](script_file.md)（script_no FK）

## 相关接口

- [ScriptProcessDataController](ScriptProcessDataController.md)
