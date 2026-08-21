# script_bind_config

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 其他 |
| 关联 Mapper | ScriptBindConfigMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | — | PK | 主键ID |
| eid | — | | 企业ID |
| platform | — | | 平台 |
| url | — | | URL |
| status | — | | 状态 |
| extendInfo | — | | 扩展信息 |
| createdTime | Long | | 创建时间 |
| updatedTime | Long | | 更新时间 |

## 关联关系

- 独立表，通过 eid 关联企业维度

## 相关接口

- [ScriptBindConfigController](ScriptBindConfigController.md)
