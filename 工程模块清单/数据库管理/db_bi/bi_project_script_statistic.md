# bi_project_script_statistic (db_bi)

- 用途：项目脚本统计，汇总各项目的脚本总数、自动化覆盖等。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| project_id | int(11) | 项目 ID |
| script_total | int(11) | 脚本总数 |
| auto_script_total | int(11) | 自动化脚本数 |
| stat_time | datetime | 统计时间 |

- 关联接口：[BiStatisticController](../../平台基础功能服务/07-开放接口文档/基础设施与统计/BiStatisticController.md)
