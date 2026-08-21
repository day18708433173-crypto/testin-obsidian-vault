# script_step_fast_search

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 脚本步骤 |
| 关联 Mapper | ScriptStepSearchMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | — | PK | 主键ID |
| script_no | — | | 脚本编号 |
| script_name | — | | 脚本名称 |
| steps_json | — | | 步骤JSON |
| create_time | Date | | 创建时间 |

## 关联关系

- 被引用：[script_file](script_file.md)（通过 script_no）

## 相关接口

- [ScriptStepSearchController](ScriptStepSearchController.md)
