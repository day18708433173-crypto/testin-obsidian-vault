# quartz_job — 定时任务模板主表

> 库：db_common（平台基础功能服务 testin-core 与 web/pc处理服务 real-web 共用）
> Mapper：`QuartzJobMapper` | XML：`QuartzJobMapper.xml`
> 实体：`cn.testin.realweb.pojo.dbCommon.QuartzJob`

## 表结构

| 字段 | 类型 | 说明 |
|------|------|------|
| job_id | int | PK，自增主键 |
| job_name | varchar | 任务名称（UUID） |
| job_rule | varchar | Cron 表达式 |
| job_desc | varchar | 任务描述 |
| job_content | text | JSON 格式完整任务内容（scripts/browsers/pcs/devices/tags等） |
| job_status | int | 状态：0=正常, 1=暂停, 2=删除, 3=过期, 4=完成 |
| job_type | varchar | TEMPLATE（模板）/ SCHEDULED（定时）/ 其他 |
| custom_job_rule | varchar | 自定义调度规则JSON（高级调度配置） |
| biz_code | int | 业务编码 |
| ent_id | int | 企业ID |
| project_id | int | 项目ID |
| user_id | int | 创建人ID |
| user_name | varchar | 创建人名称 |
| business_type | int | 业务类型（APP/WEB/PC） |
| delete_status | int | 删除标记：0=有效, 1=已删除 |
| created_by | int | 创建人 |
| created_time | datetime | 创建时间 |
| update_user_id | int | 最后更新人 |
| updated_by | int | 最后更新人 |
| updated_time | datetime | 最后更新时间 |
| trigger_last_time | datetime | 上次触发时间（XML mapper 使用，实体中不存在） |
| trigger_next_time | datetime | 下次触发时间（XML mapper 使用，实体中不存在） |

## 字段定义核实结论（2026-08-12 对实库验证）

已对实库执行 `SHOW CREATE TABLE db_common.quartz_job` 验证：**以 web/pc处理服务（real-web）版本为准**。平台基础功能服务（testin-core）文档原记的 `id`、`cron_expression`、`job_group`、`job_class`、`status`、`create_time`/`update_time` 等字段在实库中均不存在，属文档记录有误（实库主键为 `job_id`，cron 字段为 `job_rule`，状态字段为 `job_status`，时间字段为 `created_time`/`updated_time`）。

## 核心查询（QuartzJobMapper.xml）

| 方法 | SQL | 说明 |
|------|-----|------|
| `selectConditionalQuery` | LEFT JOIN app_info，paged | 条件分页查询（支持 app 信息） |
| `scheduleJobQuery` | WHERE delete_status=0 AND job_status=0 AND trigger_next_time<=maxNextTime ORDER BY job_id ASC LIMIT pagesize | 调度器取待执行 job |
| `scheduleUpdate` | UPDATE trigger_last_time, trigger_next_time, job_status | 调度器执行后回写 |

## 代码引用（web/pc处理服务）

| 调用者 | 操作 | 场景 |
|--------|------|------|
| `TemplateService` | selectPage / insert / update | 模板 CRUD |
| `BaseQuartz` | selectPage / update / insert | 调度模板列表/更新/新增 |
| `AppQuartz` | insert / selectConditionalQuery | APP 模板管理 |
| `WebQuartz` | insert / update | Web 模板管理 |
| `McPcQuartz` | insert（@Transactional）/ update / delete | PC 模板管理 |
| `QuartzJobServiceImpl` | selectList / selectCount / update | 调度注册/管理 |
| `ToQuartzJobStrategy` | update | 模板更新策略 |
| `TaskTemplateDeleteStrategyService` | selectById | 操作日志记录 |
| `TaskTemplateUpdateStrategyService` | selectById | 操作日志记录 |

## 使用方说明

### web/pc处理服务（real-web）
- 用途：定时任务模板主表，存储模板/定时调度定义，被 `TemplateService`、各 Quartz 服务（AppQuartz/WebQuartz/McPcQuartz）、`QuartzJobServiceImpl` 等读写（见上"代码引用"）。

### 平台基础功能服务（testin-core）
- 用途：定时任务定义，存储定时任务的名称、cron 表达式、状态等。
- 关联接口：[QuartzController](../../平台基础功能服务/07-开放接口文档/基础设施与统计/QuartzController.md)

## 相关文档

- [db_common 表索引](00-表索引.md)
- [TestTemplateController](../../web-pc处理服务/07-开放接口文档/模板管理/TestTemplateController.md)（模板 CRUD 入口）
- [quartz_job_log](quartz_job_log.md)
