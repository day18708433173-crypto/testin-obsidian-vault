---
tags: [核心链路, mermaid]
---

# 核心链路：Web 任务执行（提测 → 装配 → 下发 → 回收 → 状态机流转）

> 基于 web/pc处理服务 syy.release.z7.8.1.0 代码分析。串联用户提测到任务完成的六阶段数据流向（PC 任务同构，差异点为 browsers→pcs、pmweb_db→pmpc_db）。
>
> - **涉及入口**：ApiServlet `action=task, op=Task.create`（service-task-Task）/ `McPcTaskApi.add`、`RealWebApi.add`（定时链路）/ V3 `TaskController`（仅查询类）
> - **关键类**：`Task`（service）、`TaskServiceImpl`、`NoticeServiceImpl`（handleTaskAssembly/handleWebtaskInit）、`ScriptSummaryServiceImpl`、`scheduling.TaskApi`、`TestProcess`/`TestResult`、`TaskProcessServiceImpl`、`ReportServiceImpl`
> - **涉及存储**：MongoDB（`pmwebTaskDetail_XX`/`pmwebScriptSummary_XX`/`pmwebDeviceRunInfo_XX`/`pmwebScriptRunInfo_XX`/`pmwebReportDetail_XX`）、MQ `mq_info_notice`（db_mq）、Redis（锁/去重）

## 链路总览

```mermaid
flowchart TD
    subgraph P1[阶段1: 提测与参数校验]
        A1["ApiServlet action=task<br/>op=Task.create"] --> A2["Task.create<br/>(service/task/Task.java)"]
        A2 --> A3["TaskServiceImpl.create<br/>(TaskServiceImpl.java)"]
        A3 --> A4["bizCode/用户/手机号/邮箱/env 校验"]
        A4 --> A5["verifyBrowsers / verifyPcs<br/>脚本与脚本组校验"]
    end

    subgraph P2[阶段2: 任务落库]
        A5 --> B1["PmTaskDetail execStatus=waiting"]
        B1 --> B2[("Mongo pmwebTaskDetail_XX<br/>taskId末2位分片")]
        B2 --> B3["MQ: TASK_ASSEMBLY"]
        B2 --> B4["MQ: PUSH_PORTAL"]
    end

    subgraph P3[阶段3: 装配 TASK_ASSEMBLY]
        B3 --> C1["MqNoticeDataThread 1000ms轮询"]
        C1 --> C2["NoticeServiceImpl.handleTaskAssembly "]
        C2 --> C3["ScriptSummaryServiceImpl.initScriptSummary<br/>写 pmwebScriptSummary_XX"]
        C2 --> C4["数据源分配<br/>analysisParamAssign / ParamUtil.analysisParams"]
        C3 --> C5["MQ: WEB_TASK_INIT"]
    end

    subgraph P4[阶段4: 下发 WEB_TASK_INIT]
        C5 --> D1["handleWebtaskInit "]
        D1 --> D2["组装 contentJson<br/>reqId/devices/scripts/standard/timeout"]
        D2 --> D3["scheduling.TaskApi.init "]
        D3 --> D4[外部服务 RealScheduling](外部服务 RealScheduling.md)
    end

    subgraph P5[阶段5: 执行与过程上报]
        D4 --> E1["浏览器/PC客户端 执行脚本"]
        E1 --> E2["TestProcess.report<br/>(service/task/TestProcess.java)"]
        E2 --> E3["TaskProcessServiceImpl.report <br/>INIT→MATCH→RUNNING→RECOVER→COMPLETE"]
    end

    subgraph P6[阶段6: 结果回收与汇总]
        E1 --> F1["TestResult.report<br/>(service/task/TestResult.java)"]
        F1 --> F2["ReportServiceImpl.reportResult "]
        F2 --> F3[("Mongo pmwebReportDetail_XX")]
        F2 --> F4["MQ: SCRIPT_STAT + REPORT_STAT"]
        F4 --> F5["TaskSummaryServiceImpl.reportStat<br/>→ PmTaskSummary"]
        F5 --> E3
    end
```

---

## 阶段 1：提测与参数校验

**入口**：ApiServlet `action=task, op=Task.create` → `service/task/Task.create`（`service/task/Task.java`）→ `TaskServiceImpl.create(JSONObject)`（`business/impl/TaskServiceImpl.java`）。定时链路（`RealWebApi.add`/`McPcTaskApi.add`）与跨端任务最终也汇聚到 `ITaskService.create`。

校验项（`TaskServiceImpl.java`）：

| 校验 | 失败行为 |
|------|----------|
| `bizCode` → `BizConfigApi.get` 业务配置存在性 | 抛 `AddTaskCode.bizConfigInvaild` |
| 用户信息 `getUserInfo(eid, projectid, userid)` | 抛 `AddTaskCode.userinfoInvaild` |
| 手机号/邮箱正则（`REGEX_MOBILE`/`REGEX_EMAIL`） | 抛 `paraInvalid` |
| `standard.video` 枚举合法性 | 抛 `paraInvalid` |
| 浏览器 `verifyBrowsers`/ PC `verifyPcs` | 设备不存在/不可用抛异常 |
| envId → 环境配置查询（`getEnvJson`） | — |

同时解析任务级执行策略 `PmrealStandard`（continueOnError/video/taskGroup/preCompleteCallBack/projectGlobalTimeOut）、数据源四要素（paramSource/paramStrategy/dataDistributeType/dataSourceSelf）、重试上限 `retryMax`（负数归零）。

## 阶段 2：任务落库（Mongo 分片）

封装 `PmTaskDetail`（`execStatus=waiting`，`vhost=Config.MODULE_NODE_ID`），`ipmtaskdetaildao.insert` 写入 Mongo 分片集合（taskId 末 2 位 → `pmwebTaskDetail_00~99`；`pt` 前缀写 pmpc_db）。随后发两条 MQ 通知（`TaskServiceImpl.java`）：

```java
// TaskServiceImpl.java（TASK_ASSEMBLY，带延迟）
assemblyNotice.setType(NoticeConfig.InfoNoticeType.TASK_ASSEMBLY.getValue());
assemblyNotice.setPublishtime(InfoNoticeType.TASK_ASSEMBLY.getDelaytime() + System.currentTimeMillis());
assemblyNotice.setExpiretime(InfoNoticeType.TASK_ASSEMBLY.getValidPeriod() + System.currentTimeMillis());
assemblyNotice.setNoticemark(taskid);
```

- `TASK_ASSEMBLY` — 延迟装配（脚本详情填充 + 数据源分配）；恒生渠道（`hundsun` 字段）走同步 `assembleAndInitTask`直接装配+下发
- `PUSH_PORTAL` — 门户进度推送（taskCreate=1）
- 测试计划渠道（`extendedChannel=TESTIN_TEST_PLAN`）跳过 TASK_ASSEMBLY，由计划侧驱动
- `executeRecordTaskId` 场景（计划重测）走 `testPlanV3Api.resetTask` 分支直接返回

**写入集合**：`pmwebTaskDetail_XX`/`pmpcTaskDetail_XX`、`mq_info_notice`。

## 阶段 3：装配（TASK_ASSEMBLY）

`MqNoticeDataThread`（1000ms 轮询，见 [横切-后台线程全景](横切-后台线程全景.md)）捞起通知 → `NoticeServiceImpl.handleTaskAssembly`（`NoticeServiceImpl.java`）：

1. `ScriptSummaryServiceImpl.initScriptSummary(taskDetail)` — 展开脚本/脚本组为脚本汇总树，写 `pmwebScriptSummary_XX`，UUID 关联 subSubTaskId；`ScriptException` 且重试 >10 次置非法
2. 数据源分配：`params`（依赖脚本参数）走 `ParamUtil.analysisParams`，`paramSource`（数据源）走 `analysisParamAssign`，结果写回 `PmTaskDetail`
3. 数据驱动执行标准（DATA/RETENTION/REPLACE）时按 rowId 预分配脚本数据
4. 发 `WEB_TASK_INIT` 通知

**写入集合**：`pmwebScriptSummary_XX`、`pmwebTaskDetail_XX`（update）、`mq_info_notice`。

## 阶段 4：调度下发（WEB_TASK_INIT → RealScheduling）

`NoticeServiceImpl.handleWebtaskInit`（`NoticeServiceImpl.java`）组装下发报文，调用远程调度服务：

- `reqId = PmTaskDetail.generateReqId(userid, taskid, createtime)` 防重
- devices：`wt` → browsers 数组；`pt` → pcs 数组
- scripts：脚本 JSON；数据驱动时 `dataDrivenScriptProcessing` 展开
- standard：合并业务配置 `execStandards` 与用户自定义（continueOnError/applyForExecution/cleanData/maxWaitingTime/video/taskGroup），缺省超时取 `TimeoutApi.getTimeoutConfig`（web/pc 分别取）
- `matchSingleDevice`（数据驱动同脚本同机）、`taskReleaseTimePeriodsList`（下发时段控制）

```java
// NoticeServiceImpl.java
TaskApi taskapi = (TaskApi) SpringHelper.getBean("scheduling.TaskApi");
...
Integer initResult = taskapi.init(contentJson);   // → RealScheduling /v3/device_task/*
```

`taskapi.init` 返回 >0 即下发成功，RealScheduling 负责浏览器/PC 客户端的设备匹配与子任务（subtaskid/subsubtaskid）派发。

## 阶段 5：执行与过程上报（状态机驱动）

执行端通过 ApiServlet `action=task, op=TestProcess.report` 上报过程事件 → `service/task/TestProcess.report`（`service/task/TestProcess.java`）→ `TaskProcessServiceImpl.report(action, content)`（`business/impl/TaskProcessServiceImpl.java`），驱动五阶段状态机：

```
INIT → MATCH → RUNNING → COMPLETE
              ↘ RECOVER ↗
```

- **INIT**：批量插入 `PmDeviceRunInfo`/`PmScriptRunInfo`（100 条/批），`execStatus=ing`，发 `TASK_INIT_NOTICE` + `PUSH_PORTAL`
- **MATCH**：设备 RUNNING + `startExecTime`，CICC 覆盖率清除
- **RUNNING** / **RECOVER**：脚本运行标记 / 设备回等待重匹配
- **COMPLETE**：见阶段 6

完整状态机见 [横切-任务状态机](横切-任务状态机.md)。

**写入集合**：`pmwebDeviceRunInfo_XX`、`pmwebScriptRunInfo_XX`、`pmwebTaskDetail_XX`。

## 阶段 6：结果回收与任务闭环

执行结果经 ApiServlet `action=task, op=TestResult.report` → `service/task/TestResult.report`（`service/task/TestResult.java`）→ `ReportServiceImpl.reportResult(TaskResult, fileJson)`（`business/impl/ReportServiceImpl.java`）：

1. 分布式锁 `LockType.ReportResultHandle`（resultCode=0 锁 taskid，否则锁 subsubtaskid）
2. `parseNormalResult` / `parseIllegalResult` 解析结果；errorMsg/OCR 文本匹配错误原因规则（`ErrorCauseTypeV3Api.getErrorCauseMathRule`）
3. 关联 `PmScriptRunInfo`（scriptNo/数据 UUID）与 `PmScriptSummary`，写 `pmwebReportDetail_XX`
4. 发 `SCRIPT_STAT`（脚本统计）+ `REPORT_STAT`（报告统计，`ReportServiceImpl.java`）→ `TaskSummaryServiceImpl.reportStat` 汇总 `PmTaskSummary`

任务级闭环在状态机 COMPLETE 阶段（`TaskProcessServiceImpl`）：`analysisCategory` 设备结果分类汇总 → `analysisTaskResult`计算 PASS/FAIL 写回 taskDetail → 批量发送完成通知：

| MQ 类型 | 去向 |
|---------|------|
| `SEND_MSG` | 钉钉/飞书等渠道消息 |
| `FINISH_EMAIL` | 完成邮件（含分享报告链接） |
| `FINISH_NOTICE` | 机器人通知 |
| `TASK_CALLBACK` | 用户 HTTP 回调 |
| `TASK_COMPLETE_NOTICE` | 测试计划完成通知 |
| `PUSH_PORTAL` | 门户完成推送 |

## 异常与边界

| 场景 | 处理 |
|------|------|
| 装配失败 | MQ 重试（失败累加 execNum，>10 次且超 1 小时置非法）；`ScriptException` >10 次直接置非法 |
| 任务取消 | `Task.cancel`（Task.java）→ `TaskServiceImpl.cancel`；跨端未初始化取消走 `CROSS_CANCEL` |
| 暂停/恢复 | `TaskServiceImpl.pause`/`resume` 仅翻转 execStatus（ing/web_pause）+ 发 PUSH_PORTAL；`execute` 仅对测试计划渠道（TESTIN_TEST_PLAN）重发 TASK_ASSEMBLY |
| 补测/重测 | `repeatTest`发 `WEB_REPEAT_TEST`；`McPcQuartz.retest`按失败状态过滤脚本重封装 |
| 状态覆盖保护 | `modifyTaskDetail`/`modifyDeviceRunInfo` 用 `LockUtil`+`synchronized` 防止已完成被中间态覆盖 |
| 结果重复上报 | `ReportResultHandle` 锁 + taskid 维度串行化 |
| 任务超时 | `projectGlobalTimeOut`（用户配置或 RealCfg 默认）随下发报文给 RealScheduling 控制 |

## 关键代码位置

| 文件 | 作用 |
|------|------|
| `service/task/Task.java` | create/cancel/repeatTest/execute 入口 |
| `business/impl/TaskServiceImpl.java` | create 主流程 / Mongo 落库 / 通知发送 / 同步装配下发 |
| `business/impl/NoticeServiceImpl.java` | handleTaskAssembly / handleWebtaskInit / TaskApi.init |
| `service/task/TestProcess.java` | 过程上报入口 |
| `business/impl/TaskProcessServiceImpl.java` | 状态机 / COMPLETE 汇总 / analysisTaskResult |
| `service/task/TestResult.java` | 结果上报入口 |
| `business/impl/ReportServiceImpl.java` | reportResult / REPORT_STAT 通知 |

## 相关文档

- [专题索引](专题-索引.md)
- [横切-任务状态机](横切-任务状态机.md)
- [横切-通知系统](横切-通知系统.md)
- [核心链路-定时任务执行](核心链路-定时任务执行.md)
- [核心链路-报告生成](核心链路-报告生成.md)
- [service-task-Task](../07-开放接口文档/其他ApiServlet/service-task-Task.md)
- [service-task-TestProcess](../07-开放接口文档/其他ApiServlet/service-task-TestProcess.md)
- [service-task-TestResult](../07-开放接口文档/其他ApiServlet/service-task-TestResult.md)
- [service-RealWebApi](../07-开放接口文档/其他ApiServlet/service-RealWebApi.md)
- [TaskController](../07-开放接口文档/任务管理/TaskController.md)
- [PmTaskDetail](../../数据库管理/web-pc处理服务-mongo/PmTaskDetail.md)
