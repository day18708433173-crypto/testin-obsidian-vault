# suite_group_script

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 套件管理 |
| 关联 Mapper | SuiteGroupScriptMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| suiteId | — | PK | 套件ID |
| groupId | — | PK | 脚本组ID |
| status | — | | 状态 |
| createtime | Long | | 创建时间 |
| updatetime | Long | | 更新时间 |

## 关联关系

- 引用：[suite_info](suite_info.md)（suiteId FK）
- 引用：[script_group](script_group.md)（groupId FK）

## 相关接口

- [SuiteGroupScriptController](SuiteGroupScriptController.md)
