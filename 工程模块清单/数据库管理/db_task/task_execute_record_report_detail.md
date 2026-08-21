# task_execute_record_report_detail (db_task)

- 用途：该表记录了task_execute_record_report中的一些大字段信息，记录完后对应的字段不会在进行修改。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| task_execute_record_id | int | 关联的任务id字段 |
| task_execute_record_report_id | bigint | 关联的执行记录报告id |
| script_param | varchar | 脚本执行时使用的参数以及依赖脚本使用的参数 |
| pre_param | varchar | 上位机执行的执行前参数 |
| post_param | varchar | 上位机执行的执行后参数 |
| step_result | varchar | 上位机执行的每个步骤结果信息 |
| result_detail | varchar | 保存上位机上报过程中返回的各种数据，url存储 |
| script_step_detail | varchar | 脚本以及子脚本步骤数量记录 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |
| video_url | varchar | 视频链接 |

- 关联接口：
  - [TaskExecuteRecordReportController](../../任务管理服务/07-开放接口文档/任务报告/TaskExecuteRecordReportController.md)

- 关联表：
  - 主表 [task_execute_record_report](task_execute_record_report.md)
