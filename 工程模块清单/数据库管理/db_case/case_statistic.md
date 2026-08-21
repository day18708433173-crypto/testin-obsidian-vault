# case_statistic (db_case)

- 用途：用例统计，按周/月统计项目的用例总数和自动化覆盖率。
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint(20) | 主键 |
| project_id | int(11) | 项目 ID |
| year_month_week | varchar(10) | 年月周 |
| case_total | int(11) | 用例总数 |
| auto_case_total | int(11) | 自动化用例数 |
| no_auto_case_total | int(11) | 非自动化用例数 |
| start_time | date | 统计周期开始 |
| end_time | date | 统计周期结束 |
| statistic_time | datetime | 统计时间 |

- 关联接口：[CaseStatisticController](../../平台基础功能服务/07-开放接口文档/测试用例/CaseStatisticController.md)
