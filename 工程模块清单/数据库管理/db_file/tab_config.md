# tab_config

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 其他 |
| 关联 Mapper | TabConfigMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | — | PK | 主键ID |
| projectId | — | | 项目ID |
| eid | — | | 企业ID |
| tab_name | — | | Tab名称 |
| url_unique | — | | 唯一URL |
| createtime | Long | | 创建时间 |
| updatetime | Long | | 更新时间 |
| status | — | | 状态 |
| creator | — | | 创建人 |

## 关联关系

- 独立表，通过 projectId、eid 关联项目维度

## 相关接口

- [TabConfigController](TabConfigController.md)
