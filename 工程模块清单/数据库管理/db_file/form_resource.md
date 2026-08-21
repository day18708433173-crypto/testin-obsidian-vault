# form_resource

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 其他 |
| 关联 Mapper | FormResourceMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | — | PK | 主键ID |
| name | — | | 资源名称 |
| type | — | | 资源类型 |
| form_type | — | | 表单类型 |
| field_name | — | | 字段名 |
| form_placeholder | — | | 表单占位符 |
| target | — | | 目标 |
| status | — | | 状态 |
| create_time | Long | | 创建时间 |
| creator | — | | 创建人 |
| update_time | Long | | 更新时间 |
| eid | — | | 企业ID |
| projectid | — | | 项目ID |
| ai_type | — | | AI类型 |

## 关联关系

- 独立表，通过 eid、projectid 关联项目维度

## 相关接口

- [FormResourceController](FormResourceController.md)
