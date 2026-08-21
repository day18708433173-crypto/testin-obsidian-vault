# job_task_relation — 定时任务与任务关联表

> 库：db_common（平台基础功能服务 testin-core 与 web/pc处理服务 real-web 共用）
> Mapper：`JobTaskRelationMapper` | XML：`JobTaskRelationMapper.xml`
> 实体：`cn.testin.realweb.pojo.dbCommon.JobTaskRelation`

## 表结构

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | PK，自增主键 |
| ent_id | int | 企业ID |
| project_id | int | 项目ID |
| job_id | int | 关联 quartz_job.job_id |
| task_id | varchar | 执行生成的任务ID |
| task_type | int | 任务类型 |
| delete_status | int | 删除标记：0=有效, 1=已删除 |
| created_by | int | 创建人 |
| created_time | datetime | 创建时间 |
| updated_by | int | 更新人 |
| updated_time | datetime | 更新时间 |

## 字段定义核实结论（2026-08-12 对实库验证）

已对实库执行 `SHOW CREATE TABLE db_common.job_task_relation` 验证：**字段集以 web/pc处理服务（real-web）版本为准**（`project_id`/`task_type`/`delete_status`/`created_by`/`updated_by` 均真实存在），但两处类型以实库修正：主键 `id` 实为 **bigint(20)**（testin-core 记对了类型）；`task_id` 实为 **varchar(32)**（real-web 记对了类型，testin-core 记的 int(11) 有误）。时间字段为 `created_time`/`updated_time`。

## 核心查询（JobTaskRelationMapper.xml）

| 方法 | SQL | 用途 |
|------|-----|------|
| `getDayTaskCount(jobId)` | COUNT WHERE job_id=? AND DATE_FORMAT(created_time,...)=DATE_FORMAT(NOW(),...) | 日执行次数限制 |
| `getLastOneTaskRelation(jobId)` | 最新一条 WHERE job_id=? AND DATE(created_time)=CURDATE() | 最近执行记录 |

## 代码引用（web/pc处理服务）

| 调用者 | 操作 | 场景 |
|--------|------|------|
| `BaseQuartz.execAdvancedJob` | insert + selectCount | 高级调度（日频/总频限制） |
| `WebQuartz.execAdvancedJob` | 同 BaseQuartz | Web 调度 |
| `McPcQuartz.execAdvancedJob` | 同 BaseQuartz | PC 调度 |

## 使用方说明

### web/pc处理服务（real-web）
- 用途：`BaseQuartz` 的高级调度模式（`execAdvancedJob`）使用，用于 job 日频/总频限制和最近一次任务记录。

### 平台基础功能服务（testin-core）
- 用途：定时任务与测试任务的关联，记录哪些测试任务由定时任务触发。
- 关联接口：[QuartzController](../../平台基础功能服务/07-开放接口文档/基础设施与统计/QuartzController.md)、[TaskController](../../平台基础功能服务/07-开放接口文档/任务管理/TaskController.md)

## 相关文档

- [db_common 表索引](00-表索引.md)
- [quartz_job](quartz_job.md) — 外键 job_id
- [quartz_job_log](quartz_job_log.md)
