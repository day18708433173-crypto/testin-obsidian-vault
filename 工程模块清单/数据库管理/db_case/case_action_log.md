# case_action_log (db_case)

- 用途：用例操作日志，记录新增/修改/删除用例及步骤的变更历史。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int(11) | 主键 |
| case_id | int(11) | 用例 ID |
| type | int(1) | 操作类型（1=新增 2=修改 3=删除 4=新增步骤 5=移除步骤 6=调整顺序 7=修改步骤 8=online保存 9=目录删除） |
| change_details_json | text | 操作详情 JSON |
| create_time | datetime | 创建时间 |
| create_user_id | int(11) | 创建人 |

- 关联接口：[CaseActionLogController](../../平台基础功能服务/07-开放接口文档/测试用例/CaseActionLogController.md)
