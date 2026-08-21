# voice_library

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 语音管理 |
| 关联 Mapper | VoiceLibraryMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | — | PK | 语音库ID |
| eid | — | | 企业ID |
| project_id | — | | 项目ID |
| lib_name | — | | 语音库名称 |
| lib_tag | — | | 语音库标签 |
| lib_desc | — | | 语音库描述 |
| createTime | Date | | 创建时间 |
| updateTime | Date | | 更新时间 |

## 关联关系

- 被引用：[voice_content_detail](voice_content_detail.md)（library_id FK）

## 相关接口

- 语音/DBC/Box 状态相关接口已下线，接口文档已于 2026-08-12 移除
