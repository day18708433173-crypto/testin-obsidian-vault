# box_status_info

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 其他 |
| 关联 Mapper | BoxStatusInfoMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | — | PK | 主键ID |
| name | — | | 名称 |
| value | — | | 值 |
| create_time | Long | | 创建时间 |
| project_id | — | | 项目ID |
| eid | — | | 企业ID |

## 关联关系

- 独立表，通过 eid、project_id 关联项目维度

## 相关接口

- 语音/DBC/Box 状态相关接口已下线，接口文档已于 2026-08-12 移除
