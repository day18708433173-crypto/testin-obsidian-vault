# script_tag

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 标签 |
| 关联 Mapper | ScriptTagMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | Long | PK | 主键ID |
| script_no | — | | 脚本编号 |
| tag_name | — | | 标签名称 |
| create_time | Date | | 创建时间 |
| update_time | Date | | 更新时间 |

## 关联关系

- 引用：[script_file](script_file.md)（script_no FK）

## 相关接口

- [ScriptTagController](ScriptTagController.md)
