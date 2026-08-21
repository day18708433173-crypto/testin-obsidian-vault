# bi_script_dot (db_bi)

- 用途：脚本打点统计，记录真机脚本的执行次数、成功率等。
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
| script_total | bigint(20) | 脚本总数 |
| script_exec_total | bigint(20) | 脚本执行总数 |

- 关联接口：[BiStatisticController](../../平台基础功能服务/07-开放接口文档/基础设施与统计/BiStatisticController.md)
