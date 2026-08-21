# app_info — 应用信息表

> 库：db_common | Mapper：`AppInfoMapper` | 实体：`cn.testin.realweb.pojo.dbCommon.AppInfo`

## 表结构

| 字段 | 类型 | 说明 |
|------|------|------|
| app_id | int | PK，自增主键 |
| job_id | int | 关联 quartz_job.job_id |
| app_name | varchar | 应用名称 |
| app_version | varchar | 应用版本 |
| suite_id | int | 套件ID |
| suite_name | varchar | 套件名称 |
| package_id | int | 包ID |
| package_name | varchar | 包名 |
| syspf_id | int | 系统平台ID |
| delete_status | int | 删除标记：0=有效, 1=已删除 |
| created_by | int | 创建人 |
| created_time | datetime | 创建时间 |
| updated_by | int | 更新人 |
| updated_time | datetime | 更新时间 |

## 核心查询（QuartzJobMapper.xml 联查）

```sql
-- selectConditionalQuery: LEFT JOIN app_info ai ON qj.job_id = ai.job_id
SELECT qj.job_id, qj.job_desc, qj.user_name, qj.created_time,
       qj.job_rule, ai.suite_name, ai.app_name, ai.syspf_id
FROM quartz_job qj
LEFT JOIN app_info ai ON qj.job_id = ai.job_id
```

## 代码引用

| 调用者 | 操作 | 场景 |
|--------|------|------|
| `AppQuartz.add` | insert | APP模板创建 |
| `AppQuartz.update` | update | APP模板更新 |
| `AppInfoServiceImpl` | CRUD | 通用CRUD |

## 相关文档

- [db_common 表索引](00-表索引.md)
- [quartz_job](quartz_job.md) — 外键 job_id
