# script_third_relation

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 脚本关系 |
| 关联 Mapper | ScriptThirdRelationMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | — | PK | 主键ID |
| script_no | — | | 脚本编号 |
| third_func_no | — | | 第三方功能号 |
| status | — | | 状态 |
| createtime | Long | | 创建时间 |
| updatetime | Long | | 更新时间 |
| third_user_id | — | | 第三方用户ID |
| third_user_name | — | | 第三方用户名 |
| third_project_id | — | | 第三方项目ID |

## 关联关系

- 引用：[script_file](script_file.md)（script_no FK）

## 相关接口

- [ScriptThirdRelationController](ScriptThirdRelationController.md)
