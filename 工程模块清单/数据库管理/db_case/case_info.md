# case_info (db_case)

- 用途：测试用例主表，存储用例名称、等级、版本、状态等核心信息。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| eid | int(11) | 企业 ID |
| project_id | int(11) | 项目 ID |
| case_uuid | varchar(50) | UUID |
| case_name | varchar(60) | 用例名称 |
| case_level | int(2) | 用例等级（P0 最高） |
| case_purpose | varchar(60) | 测试目的 |
| case_version | varchar(100) | 用例版本 |
| case_dir_id | int(10) | 目录 ID |
| case_remark | varchar(300) | 备注 |
| case_status | int(1) | 状态（1=待评审 2=待设计 3=设计中 4=已完成 5=已废弃） |
| case_check_status | int(1) | 脚本检查状态 |
| create_user_id | int(11) | 创建人 |
| update_user_id | int(11) | 更新人 |
| status | int(1) | 数据状态（0=无效 1=有效） |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |

- 关联接口：[CaseInfoController](../../平台基础功能服务/07-开放接口文档/测试用例/CaseInfoController.md)
