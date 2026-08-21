# script_dir

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 目录管理 |
| 关联 Mapper | ScriptDirMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| script_dir_id | — | PK | 目录ID |
| eid | — | | 企业ID |
| projectid | — | | 项目ID |
| parent_dir_id | — | | 父目录ID |
| script_dir_name | — | | 目录名称 |
| script_dir_creator_userid | — | | 创建人ID |
| script_dir_updater_userid | — | | 更新人ID |
| script_dir_status | — | | 目录状态 |
| script_dir_version | — | | 目录版本 |
| createtime | Long | | 创建时间 |
| updatetime | Long | | 更新时间 |
| is_delete | — | | 是否删除 |
| dir_type | — | | 目录类型 |
| dir_order | — | | 目录排序 |

## 关联关系

- 被引用：[script_dir_child](script_dir_child.md)（script_dir_id FK）
- 被引用：[script_file](script_file.md)（scriptDirId FK）

## 相关接口

- [ScriptDirController](../../脚本服务/07-开放接口文档/目录管理/ScriptDirController.md)
