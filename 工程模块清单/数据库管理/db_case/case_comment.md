# case_comment (db_case)

- 用途：用例评论，支持在对应用例下添加讨论信息。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| case_id | int(11) | 用例 ID |
| case_comment | text | 评论内容 |
| create_user_id | int(10) | 创建人 |
| update_user_id | int(10) | 更新人 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |
| status | int(1) | 状态（0=无效 1=有效） |

- 关联接口：[CaseCommentController](../../平台基础功能服务/07-开放接口文档/测试用例/CaseCommentController.md)
