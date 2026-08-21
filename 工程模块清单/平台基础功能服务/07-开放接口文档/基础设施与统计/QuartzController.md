# QuartzController -- 定时任务模版管理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/quartz/QuartzController.java`
> 类级路由：`/v3/core/quartz`
> Service 接口：`cn.testin.business.interfaces.quartz.IQuartzService`
> 实现类：`cn.testin.business.impl.quartz.QuartzServiceImpl`（`@Qualifier("quartzServiceImpl")`）
> 业务：定时任务模版的分页查询列表，以及数据源初始化。

## 接口列表总表

| 方法 | 路径 | 方法名 | 说明 | 操作日志 |
|---|---|---|---|---|
| GET | `/v3/core/quartz/list_scheduled_job` | listScheduledJob | 分页获取任务模版列表 | 无 |
| POST | `/v3/core/quartz/dataSource` | dataSource | 数据源初始化（运维用） | 无 |

统一响应包装：`ResponseResult<T>`；分页用 `BasePageListResponseDTO`；写操作用 `BaseDataResultDTO`。

---

## 1. GET /v3/core/quartz/list_scheduled_job -- 任务模版列表

### 入口

`QuartzController.listScheduledJob(QuartzJobInfoRequestDTO quartzJobInfoRequestDTO)` -- QuartzController.java（`@UnderlineToCamel`）

### 请求参数（QuartzJobInfoRequestDTO，Query 绑定）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 是 | 项目ID |
| jobName | String | 否 | 任务模版名称（模糊匹配） |
| jobStatus | Integer | 否 | 任务状态过滤 |
| taskType | Integer | 否 | 任务类型过滤（1 APP / 3 Web / 5 PC / CASE） |
| dirId | Integer | 否 | 目录ID过滤（通过 dir_quartz_job 关联） |
| page | Integer | 否 | 页码，为 null 或 <=0 取 `PAGE_DEFAULT` |
| pageSize | Integer | 否 | 页大小，为 null 或 <=0 取 `PAGE_SIZE_DEFAULT` |

### 响应结构

`ResponseResult<BasePageListResponseDTO<QuartzJobInfoResponseDTO>>`。

### 返回参数（QuartzJobInfoResponseDTO 元素结构）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 分页数据对象 |
| data.list | Array\<QuartzJobInfoResponseDTO\> | 任务模版列表 |
| data.list[].id | Integer | 模版主键 |
| data.list[].jobId | Long | Quartz Job ID |
| data.list[].jobName | String | 模版名称 |
| data.list[].jobRule | String | 任务规则 |
| data.list[].jobStatus | Long | 任务状态 |
| data.list[].jobRemark | String | 任务备注 |
| data.list[].userId | Integer | 用户 ID |
| data.list[].userName | String | 用户名 |
| data.list[].eid | Integer | 企业 ID |
| data.list[].projectId | Integer | 项目 ID |
| data.list[].taskContent | String | 任务内容 |
| data.list[].newTaskContent | String | 新任务内容 |
| data.list[].taskDesc | String | 任务描述 |
| data.list[].appId | Integer | 应用 ID |
| data.list[].appName | String | 应用名 |
| data.list[].appVersion | String | 应用版本 |
| data.list[].pkgId | Integer | 安装包 ID |
| data.list[].packageName | String | 包名 |
| data.list[].bizCode | Integer | 业务码 |
| data.list[].syspfId | Integer | 系统平台 ID |
| data.list[].channelId | String | 渠道 ID |
| data.list[].suiteId | Integer | 套件 ID |
| data.list[].jobType | Integer | 任务类型 |
| data.list[].customJobRule | String | 自定义任务规则 |
| data.list[].createTime | Long | 创建时间 |
| data.list[].updateTime | Long | 更新时间 |
| data.list[].relations | Array\<TaskRecordsDTO\> | 关联记录（元素含 id/planInfoName/planDeviceStatus） |
| data.list[].jobDesc | String | 任务描述 |
| data.list[].jobContent | String | 任务内容 |
| data.list[].entId | Integer | 企业 ID（兼容字段） |
| data.list[].businessType | Integer | 业务类型 |
| data.list[].deleteStatus | Integer | 删除状态 |
| data.list[].createdBy | String | 创建人 |
| data.list[].updatedBy | String | 更新人 |
| data.list[].effectiveDeviceTotal | Integer | 有效设备总数 |
| data.list[].taskType | Integer | 任务类型（1 APP / 3 Web / 5 PC） |
| data.list[].taskName | String | 任务名 |
| data.list[].taskTemplateStatus | Integer | 模版状态 |
| data.list[].templateType | Integer | 模版类型 |
| data.list[].cronExpression | String | Cron 表达式 |
| data.list[].cronRule | String | Cron 规则 |
| data.list[].createUserId | Integer | 创建人 ID |
| data.list[].updateUserId | Integer | 更新人 ID |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页条数 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总记录数 |

### 实现意图

Controller 层补齐分页默认值（`PAGE_DEFAULT` / `PAGE_SIZE_DEFAULT`）后委托 `QuartzServiceImpl.listScheduledJob`，查表返回分页列表。

### 调用链

```
QuartzController.listScheduledJob
└─ QuartzServiceImpl.listScheduledJob
   → quartz_job 相关表分页查询
```

### 涉及表

| 表 | 操作 |
|---|---|
| quartz_job 系列表（quartz_job / dir_quartz_job） | 读（分页条件查询） |

---

## 2. POST /v3/core/quartz/dataSource -- 数据源初始化

### 入口

`QuartzController.dataSource()` -- QuartzController.java

### 请求参数

无。

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 初始化影响行数（Long 类型）。

### 实现意图

运维端点：调用 `QuartzServiceImpl.dataInit()` 执行数据源初始化逻辑（可能涉及 quartz 表的预设数据灌入或迁移）。具体逻辑见 Service 实现。

### 调用链

```
QuartzController.dataSource
└─ QuartzServiceImpl.dataInit
```

---

## 备注

- 本 Controller 是定时任务模版（Quartz Job）的管理入口，与物理 Quartz 调度引擎的操作不同。
- 任务模版通过 `dir_quartz_job` 表与目录关联，目录树查询见 [DirInfoController](DirInfoController.md)，关联关系管理见 [DirQuartzJobController](DirQuartzJobController.md)。
- `QuartzJobInfoRequestDTO` 定义见 `cn.testin.dto.request.task` 包。
- `dataSource` 为无参 POST，不设鉴权注解，属内部运维端点，需注意网关暴露范围。

相关文档：[00-分支索引](00-分支索引.md)
