# quartz_job_log — 定时任务执行日志

> 库：db_common（平台基础功能服务 testin-core 与 web/pc处理服务 real-web 共用）
> Mapper：`QuartzJobLogMapper`
> 实体：`cn.testin.realweb.pojo.dbCommon.QuartzJobLog`

## 表结构

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | PK，自增主键 |
| job_id | int | 关联 quartz_job.job_id |
| task_id | varchar | 执行生成的任务ID |
| request_content | text | 请求内容JSON |
| response_content | text | 响应内容JSON |
| created_by | int | 创建人 |
| created_time | datetime | 创建时间 |
| updated_by | int | 更新人 |
| updated_time | datetime | 更新时间 |

## 字段定义核实结论（2026-08-12 对实库验证）

已对实库执行 `SHOW CREATE TABLE db_common.quartz_job_log` 验证：**以 web/pc处理服务（real-web）版本为准**。平台基础功能服务（testin-core）文档原记的 `execute_time`、`execute_result`、`error_msg`、`create_time` 等字段在实库中均不存在，属文档记录有误（实库主键 `id` 为 int 自增，日志内容为 `task_id` + `request_content`/`response_content` 请求响应报文）。

## 核心查询

| 方法 | 说明 |
|------|------|
| `QuartzJobLogMapper.selectList` | 按 task_id 查询 |
| `QuartzJobLogMapper.delete` | 按 task_id 删除 |
| `QuartzJobMapper.listTaskIdByJobId` | 联查：按 jobId 获取 task_id 列表 |

## 代码引用（web/pc处理服务）

| 调用者 | 操作 | 场景 |
|--------|------|------|
| `RealWebApi.add` | insert | Web 任务创建时记录 |
| `McPcTaskApi.add` | insert | PC 任务创建时记录 |
| `QuartzLogService.removeByTaskId` | selectList + delete | [QuartzLogController](../../web-pc处理服务/07-开放接口文档/基础设施与统计/QuartzLogController.md) 删除 |
| `QuartzJobLogServiceImpl` | CRUD | 通用日志查询 |
| `BaseQuartz.getTaskIdProdByQuartzJob` | 联查 | 按 jobId 获取历史 taskId |

## 使用方说明

### web/pc处理服务（real-web）
- 用途：记录定时任务每次触发后生成的任务（task_id）及请求/响应内容，Web/PC 任务创建时写入。

### 平台基础功能服务（testin-core）
- 用途：定时任务执行日志，记录每次任务触发和执行结果。
- 关联接口：[QuartzController](../../平台基础功能服务/07-开放接口文档/基础设施与统计/QuartzController.md)

## 相关文档

- [db_common 表索引](00-表索引.md)
- [quartz_job](quartz_job.md) — 外键 job_id
- [QuartzLogController](../../web-pc处理服务/07-开放接口文档/基础设施与统计/QuartzLogController.md)
