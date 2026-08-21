# execute_record_task_case_detail (db_plan)

- 用途：用例执行的详细结果，记录每个数据标识符下用例的通过/失败/跳过状态。
- 主要字段：

| 字段                          | 类型           | 说明       |
| --------------------------- | ------------ | -------- |
| id                          | bigint(20)   | 主键       |
| execute_record_id           | bigint(20)   | 执行记录 ID  |
| execute_record_task_id      | bigint(20)   | 执行任务 ID  |
| execute_record_task_case_id | bigint(20)   | 用例记录 ID  |
| data_identifier_id          | varchar(255) | 数据标识符 ID |
| execute_status              | int(11)      | 执行状态     |
| execute_result              | int(11)      | 执行结果     |
| execute_start_time          | datetime     | 开始时间     |
| execute_end_time            | datetime     | 结束时间     |
| error_msg                   | text         | 错误信息     |
| create_time                 | timestamp    | 创建时间     |
| is_delete                   | int(11)      | 是否删除     |

- 关联接口：[ExecuteRecordTaskController](../../平台基础功能服务/07-开放接口文档/测试计划/ExecuteRecordTaskController.md)
