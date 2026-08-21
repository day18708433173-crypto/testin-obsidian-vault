# script_dir_child

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 目录管理 |
| 关联 Mapper | ScriptDirChildMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | — | PK | 主键ID |
| script_no | — | | 脚本编号 |
| eid | — | | 企业ID |
| projectid | — | | 项目ID |
| script_dir_id | — | | 目录ID |
| updater_userid | — | | 更新人ID |
| updatetime | Long | | 更新时间 |

## 关联关系

- 引用：[script_file](script_file.md)（script_no FK）
- 引用：[script_dir](script_dir.md)（script_dir_id FK）

## 相关接口

- [ScriptDirChildController](ScriptDirChildController.md)
