# service-RealWebApi — Web 任务创建与详情

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/task/RealWebApi.java`（@Component）
> 类型：本地业务逻辑类（非远端转发）
> 本地注入：`QuartzJobLogMapper`, `ITaskService`, `ITaskDetailService`, `IReportService`, `PortalApi`, `IScriptRunInfoService`

与 [McPcTaskApi](service-McPcTaskApi.md) 为 Web/PC 对等实现，结构高度一致，区别在于设备类型（browsers vs pcs）。

## 方法列表

### 1. add — 创建 Web 任务

```java
public String add(QuartzJob quartzJob, Map params)
```

**用途**：由 `WebQuartz.execute()` 调用，创建 Web 端测试任务。

**流程**：
1. 从 `quartzJob.job_content` 提取 Web 特有参数：`browsers`（替代 pcs）、`execStandard`、`tagList`、`skipTagList`、`selectFailNoticeChannelCfgList`、`deviceOfflineConfig` 等
2. 构建 `action=task, op=Task.create` 请求体
3. 调用 `ITaskService.create(dataJson)` → 本地创建任务
4. 参数校验（要求 scripts+browsers 非空）
5. 写入 `QuartzJobLog`

**提取的字段**（与 McPcTaskApi 共同）：robotNotice, noticeChannel, minRule, maxRule, channelCfg, emails, mobiles, msgTempletId, paramSource, paramSourceName, paramStrategy, dataDistributeType, dataSourceSelf, standard, retryMax, env, matchSingleDevice, failNoticeScriptType, failNoticeScriptNum, extendedChannel, extendedChannelUrl + projectId/eid/userId/jobId/quartz marker/isManualExecution

**Web 特有字段**：`scripts`, `browsers`, `execStandard`, `tagList`, `skipTagList`, `selectFailNoticeChannelCfgList`, `deviceOfflineConfig`

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| quartzJob | QuartzJob | 是 | 定时任务对象；`jobContent` 内 scripts/browsers 必填（缺失抛 `paraInvalid`），projectId/taskDescr/bizCode/eid/userId 也需非空 |
| params | Map | 否 | 附加参数（isManualExecution=1 时覆盖 userid，可选 extendedChannel 等） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| taskId | String | 新创建任务 id（可为 null） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmTaskDetail | 写（→ITaskService.create） |
| MySQL db_common | quartz_job_log | 写 |

### 2. detail — Web 任务详情

```java
public PmTaskDetail detail(String taskId, Integer eid, Integer projectId, JSONArray scriptStatuses)
```

与 McPcTaskApi.detail 完全相同逻辑（复制代码），读取 Web 端任务详情并丰富：过滤 devices/scripts、标签名称、脚本数量计算、Portal 时间查询。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskId | String | 是 | 任务 id（`ITaskDetailService.get` 无 null 校验，null 会 NPE） |
| eid | Integer | 否 | 企业 id（仅传给 PortalApi 查询耗时） |
| projectId | Integer | 否 | 项目 id |
| scriptStatuses | JSONArray | 是 | 脚本状态过滤数组（代码直接 `isEmpty()`，为 null 会 NPE） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | PmTaskDetail | Web 任务详情（含 scripts/browsers/scriptNum/scriptTotalExecTime/sourceTagName/skipTagName 等，完整字段见 MongoDB PmTaskDetail） |

## 相关文档

- [00-分支索引](00-分支索引.md)
- [service-McPcTaskApi](service-McPcTaskApi.md)（PC 端对等实现）
- [WebQuartz](WebQuartz.md)
