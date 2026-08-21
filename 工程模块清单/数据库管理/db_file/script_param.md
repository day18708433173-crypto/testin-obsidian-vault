# script_param

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 参数与数据源 |
| 关联 Mapper | ScriptParamMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | Long | PK | 主键ID |
| projectid | — | | 项目ID |
| userid | — | | 用户ID |
| appid | — | | 应用ID |
| name | — | | 参数名称 |
| default_value | — | | 默认值 |
| descr | — | | 描述 |
| version | — | | 版本 |
| status | — | | 状态 |
| createtime | Long | | 创建时间 |
| updatetime | Long | | 更新时间 |

## 关联关系

- 被引用：[script_param_data](script_param_data.md)（通过业务关联）
- 被引用：[script_param_source](script_param_source.md)（通过业务关联）

## 相关接口

- [ScriptParamController](ScriptParamController.md)
