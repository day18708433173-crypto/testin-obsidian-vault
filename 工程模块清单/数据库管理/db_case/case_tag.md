# case_tag (db_case)

- 用途：用例标签，用于用例分类和筛选。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(20) unsigned | 主键 |
| case_id | int(11) | 用例 ID |
| tag_name | varchar(100) | 标签名称 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 关联接口：[CaseInfoController](../../平台基础功能服务/07-开放接口文档/测试用例/CaseInfoController.md)
