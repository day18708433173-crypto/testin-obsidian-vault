# service-McPcTaskApi — PC 任务创建与详情

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/task/McPcTaskApi.java`（@Component）
> 类型：本地业务逻辑类（非远端转发）
> 本地注入：`QuartzJobLogMapper`, `ITaskService`, `ITaskDetailService`, `IReportService`, `PortalApi`, `IScriptRunInfoService`

## 方法列表

### 1. add — 创建 PC 任务

```java
public String add(QuartzJob quartzJob, Map params)
```

**用途**：由 `McPcQuartz.execute()` 调用，创建 PC 端测试任务。

**流程**：
1. 从 `quartzJob.job_content` 提取参数（robotNotice, noticeChannel, minRule, maxRule, channelCfg, emails, mobiles, msgTempletId, paramSource, scripts, pcs 等）
2. 构建 `action=scheduling, op=Task.create` 请求体
3. 调用 `ITaskService.create(dataJson)` → 本地创建任务
4. 设置 `quartz` marker, 处理 `isManualExecution` 覆盖 userId
5. 校验参数（`checkCreateParams` 要求 eid/projectid/userId/bizCode/taskDescr/pcs/scripts）
6. 写入 `QuartzJobLog`（insert → quartz_job_log）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| quartzJob | QuartzJob | 是 | 定时任务对象；`jobContent` 内 eid/projectid/userid/bizCode/taskDescr/pcs/scripts 经 `checkCreateParams` 校验必填 |
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

### 2. detail — PC 任务详情

```java
public PmTaskDetail detail(String taskId, Integer eid, Integer projectId, JSONArray scriptStatuses)
```

**用途**：获取 PC 任务详情并丰富扩展信息。

**流程**：
1. `ITaskDetailService.get(taskId)` → MongoDB PmTaskDetail
2. `Task.filterDevice` → 过滤设备列表
3. `ParameterSourceApi.getTagInfoList` → 标签/skip标签名称
4. `IReportService.getScriptByResultCategory` → 按结果分类过滤脚本
5. 计算各 execStandard 的脚本数量
6. `PortalApi.getPortalTask` → 获取 scriptTotalExecTime

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
| (对象) | PmTaskDetail | PC 任务详情（含 scripts/browsers/pcs/scriptNum/scriptTotalExecTime/sourceTagName/skipTagName 等，完整字段见 MongoDB PmTaskDetail） |

**涉及表**：

| 存储 | 集合 | 操作 |
|------|------|------|
| MongoDB | PmTaskDetail | 读 |
| MongoDB | PmReportDetail | 读 |
| Remote DataSource | 标签表 | 读 |
| Remote RealPortal | portal_task | 读 |

## 相关文档

- [00-分支索引](00-分支索引.md)
- [service-RealWebApi](service-RealWebApi.md)（Web 端对等实现）
- [WebQuartz](WebQuartz.md) / [McPcQuartz](McPcQuartz.md)
