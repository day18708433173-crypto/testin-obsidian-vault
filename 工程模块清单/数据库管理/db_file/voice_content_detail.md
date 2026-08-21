# voice_content_detail

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 语音管理 |
| 关联 Mapper | VoiceContentDetailMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | — | PK | 主键ID |
| uniq_key | — | | 唯一键 |
| eid | — | | 企业ID |
| library_id | — | | 语音库ID |
| voc_url | — | | 语音文件URL |
| voc_name | — | | 语音名称 |
| voc_desc | — | | 语音描述 |
| voc_tag | — | | 语音标签 |
| voc_type | — | | 语音类型 |
| available | — | | 是否可用 |
| wake_word_default | — | | 默认唤醒词 |
| status | — | | 状态 |
| createTime | Date | | 创建时间 |
| updateTime | Date | | 更新时间 |

## 关联关系

- 引用：[voice_library](voice_library.md)（library_id FK）

## 相关接口

- 语音/DBC/Box 状态相关接口已下线，接口文档已于 2026-08-12 移除
