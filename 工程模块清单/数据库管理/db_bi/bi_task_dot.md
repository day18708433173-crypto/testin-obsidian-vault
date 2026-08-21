# bi_task_dot (db_bi)

- 用途：任务打点统计，记录真机任务的创建、执行等统计数据。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| stat_time | int(11) | 统计时间 |
| eid | int(11) | 企业 ID |
| ename | varchar(128) | 企业名称 |
| pid | int(11) | 项目 ID |
| pname | varchar(128) | 项目名称 |
| uid | int(11) | 用户 ID |
| uname | varchar(128) | 用户名称 |
| task_total | bigint(20) | 任务总数 |

- 关联接口：[BiStatisticController](../../平台基础功能服务/07-开放接口文档/基础设施与统计/BiStatisticController.md)
