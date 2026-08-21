# case_revision (db_case)

- 用途：用例版本历史，记录每次修改的详细变更。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| eid | int(11) | 企业 ID |
| project_id | int(11) | 项目 ID |
| case_id | int(11) | 用例 ID |
| create_user_id | int(11) | 创建人 |
| create_time | datetime | 创建时间 |
| case_revision_details | text | 历史详情 |

- 关联接口：[CaseActionLogController](../../平台基础功能服务/07-开放接口文档/测试用例/CaseActionLogController.md)
