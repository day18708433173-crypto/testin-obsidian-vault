# dbc_file

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 语音管理 |
| 关联 Mapper | DbcFileMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | Long | PK | 主键ID |
| project_id | — | | 项目ID |
| dbcFilePath | — | | DBC文件路径 |
| dbcFileName | — | | DBC文件名 |
| status | — | | 状态 |
| tag | — | | 标签 |
| dbcFileDesc | — | | DBC文件描述 |
| createBy | — | | 创建人ID |
| createUserName | — | | 创建人用户名 |
| createTime | Date | | 创建时间 |
| updateTime | Date | | 更新时间 |
| version | — | | 版本 |
| dbcSecretKeyPath | — | | DBC密钥文件路径 |

## 关联关系

- 独立表，通过 project_id 关联项目维度

## 相关接口

- 语音/DBC/Box 状态相关接口已下线，接口文档已于 2026-08-12 移除
