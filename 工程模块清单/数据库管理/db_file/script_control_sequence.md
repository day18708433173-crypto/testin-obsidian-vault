# script_control_sequence

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 脚本控件 |
| 关联 Mapper | ScriptControlSequenceMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| projectid | — | PK | 项目ID |
| appid | — | PK | 应用ID |
| script_control_id | — | | 控件ID |

## 关联关系

- 引用：[script_control](script_control.md)（script_control_id FK）

## 相关接口

- [ScriptControlSequenceController](ScriptControlSequenceController.md)
