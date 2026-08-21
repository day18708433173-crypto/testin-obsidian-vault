# case_script_no_sync (db_case)

- 用途：用例中脚本的同步检查记录，跟踪脚本与用例的一致性。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| script_no | int(11) | 脚本编号 |
| check_status | int(11) | 检查状态 |
| sync_count | int(11) | 同步次数 |

- 关联接口：[CaseStatisticController](../../平台基础功能服务/07-开放接口文档/测试用例/CaseStatisticController.md)
