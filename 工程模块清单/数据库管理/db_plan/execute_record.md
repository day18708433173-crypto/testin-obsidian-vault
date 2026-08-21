# execute_record (db_plan)

- 用途：计划执行记录主表，每次计划执行生成一条记录。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| project_id | int(11) | 项目 ID |
| plan_info_id | bigint(20) | 测试计划 ID |
| plan_info_type | int(11) | 测试计划类型 |
| execute_record_name | varchar(255) | 报表名称 |
| plan_info_name | varchar(255) | 计划名称快照 |
| suite_id | int(11) | 应用 ID |
| sync_cancel | int(11) | 终止同步标记 |
| pre_test_task | int(11) | 预提测完成标记 |
| report_excel_url | varchar(255) | 报表 Excel URL |
| plan_email_cfg | text | 邮件配置 |
| email_send_status | int(11) | 邮件发送状态 |
| execute_period | varchar(511) | 执行周期快照 |
| call_back_url | varchar(1023) | 回调地址 |
| create_user_id | int(11) | 创建人 |
| create_time | timestamp | 创建时间 |
| update_time | timestamp | 更新时间 |
| is_delete | int(11) | 是否删除 |

- 关联接口：[ExecuteRecordController](../../平台基础功能服务/07-开放接口文档/测试计划/ExecuteRecordController.md)
