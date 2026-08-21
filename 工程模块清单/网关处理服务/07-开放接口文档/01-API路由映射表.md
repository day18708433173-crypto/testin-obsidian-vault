---
module: proxy-openapi
type: 开放接口文档
---

# API 路由映射表

> 来源：db_mcfg 库 view_role_api 视图（实时数据）+ 代码逻辑。每条 API 经网关的认证鉴权后，按 McfgModule.httpHosts 路由到目标服务。以下按目标知识库服务分组，每个模块下列出其代理的全部接口。

## 网关路由规则回顾

```
请求 → ApiProxyServlet(V1) / ApiV2ProxyServlet(V2) / ApiV3ProxyServlet(V3)
     → commAuth（apiKey→McfgApp → mkey→McfgModule → action+op→McfgApi）
     → executeRequest:
         httpHosts 为空 → RPC (ICE, McfgModule.rpcPrefixName)
         httpHosts 不为空 → HTTP 透传 (HttpURLConnection)
```

### 请求 URL 格式

| 版本 | Servlet 映射 | URL 格式 | 示例 |
|---|---|---|---|
| V1 | `/*` | `POST https://api.pro.testin.cn/{任意路径}` Body: `{"mkey":"...", "action":"...", "op":"...", "apikey":"...", ...}` | `POST /` → `mkey=realtest, action=app, op=Task.add` |
| V2 | `/v2/*` | `POST /v2/{mkey}/{action}/{op}` Body: 直接 data JSON | `POST /v2/realtest/app/Task.add` |
| V3 | `/v3/*` `/openapi/v3/*` | `{METHOD} /v3/{mkey}/{resource}` 或 `/openapi/v3/{mkey}/{resource}` | `GET /v3/real_task/task_template` `POST /openapi/v3/fs/app/upload` |

所有 HTTP 模块均需携带 `apikey`+`mkey` 参数；needLogin=1 的接口额外需要 `sid`。
`/openapi/v3/*` 前缀的接口由 `ApiV3ProxyServlet` 处理，与 `/v3/*` 共享同一套认证鉴权逻辑。

### 转发模式说明（关键）

网关对 V3 请求有三种转发模式，由 `mcfg_api.pass_through_type` 和 `special_api_action`/`special_api_op` 共同决定：

| 模式 | pass_through_type | 判定依据 | Servlet | 转发逻辑 |
|---|---|---|---|---|
| **V1 原生** | 0 | `api_action` 为短格式（如 `app.Task.add`） | ApiProxyServlet | `executeRequest()` → RPC 或 HTTP 代理 |
| **V3 透传** | **1** | `api_action` 为 URL 格式（如 `/v3/realtest/report/steps`） | ApiV3ProxyServlet | `doPressWithPassThrough()` 直接透传原始请求到目标 HTTP 服务 |
| **V3→V1 转换** | **0** | URL 格式 api_action + 有 `special_api_action`/`special_api_op` | ApiV3ProxyServlet | `preExecuteRequest()` 将 V3 URL 转换为 V1 sendJson → `executeRequest()` |

**代码路径**（`ApiV3ProxyServlet.java`）：

```
doPress():
  preExecuteRequest():
    passThroughType==1 → return immediately（不做 V1 转换）
    passThroughType!=1 → 构建 V1 sendJson（mkey/action/op/sid/apikey）
      special_api_action 不为空 → 覆盖 action 为此值
      special_api_op 不为空 → 覆盖 op 为此值
  
  getResponseResult():
    passThroughType==1 → doPressWithPassThrough()   ← 直接透传到 HTTP 目标
    passThroughType!=1 → executeRequest()            ← 走标准 V1 代理
```

下文每个接口标注转发模式：🟢=V1原生 / 🔵=V3透传 / 🟡=V3→V1转换。

---

## 1. app处理服务（real-test）

> 模块 `realtest` → `http://testin-aio-real-test:8080`，接口文档索引：[real-test 接口索引](../../app处理服务（real-test）/07-开放接口文档/00-模块索引.md)

### V1 原生（action/op 模式，passThroughType=0，short action）

| 转发 | action | op | 说明 | needLogin | 对应文档 |
|---|---|---|---|---|---|
| 🟢 | app | Task.add | 新增测试 | 1 | [Task](../../app处理服务（real-test）/07-开放接口文档/测试计划与任务/Task.md) |
| 🟢 | app | Task.cancel | 终止测试 | 1 | 同上 |
| 🟢 | app | Task.batchCancel | 批量取消/终止 | 1 | 同上 |
| 🟢 | app | Task.detail / details | 测试任务详情 | 1 | 同上 |
| 🟢 | app | Task.repeatTest | 脚本/设备批量补测 | 1 | 同上 |
| 🟢 | app | Task.retest | 设备补测 | 1 | 同上 |
| 🟢 | app | Task.scriptRetest | 脚本补测 | 1 | 同上 |
| 🟢 | app | Task.initData | 初始化任务数据 | 1 | 同上 |
| 🟢 | app | Task.share | 任务分享 | 1 | 同上 |
| 🟢 | app | Task.sendEmail | 重新发送邮件 | 1 | 同上 |
| 🟢 | app | Task.verification | taskid/skey验证 | 1 | 同上 |
| 🟢 | app | Task.maintainReportSummary | 报告总结提交 | 1 | 同上 |
| 🟢 | app | Task.overview / userAdapt | 概况/用户信息 | 1 | 同上 |
| 🟢 | app | ScheduledJob.* | 定时计划CRUD/执行/暂停 | 1 | [ScheduledJob](../../app处理服务（real-test）/07-开放接口文档/测试计划与任务/ScheduledJob.md) |
| 🟢 | app | Safety.listSafeInfo | 梆梆安全查询 | 1 | — |
| 🟢 | app | ParamSource.assign | 根据设备分配数据源 | 0 | — |
| 🟢 | report | Report.* | 报告查询(27 op) | 1 | [ReportController](../../app处理服务（real-test）/07-开放接口文档/报告与导出/ReportController.md) / [Report](../../app处理服务（real-test）/07-开放接口文档/报告与导出/Report.md) |
| 🟢 | report | Excel.reportExcel | Excel导出 | 1 | [Excel](../../app处理服务（real-test）/07-开放接口文档/报告与导出/Excel.md) |
| 🟢 | report | Pdf.parse | PDF导出 | 1 | [Pdf](../../app处理服务（real-test）/07-开放接口文档/报告与导出/Pdf.md) |
| 🟢 | report | Qc.notify | QC通知 | 1 | — |
| 🟢 | analysis | Performance.* | 性能数据图表/导出 | 1 | — |
| 🟢 | analysis | Report.* | 错误汇总/问题分析 | 1 | — |
| 🟢 | analysis | Task.overview | 执行概况 | 1 | — |
| 🟢 | script | ScriptList.* | 脚本资源查询(7 op) | 1 | — |

### V3 透传（passThroughType=1，直接透传）

| 转发 | method | URL | 说明 | needLogin | 对应文档 |
|---|---|---|---|---|---|
| 🔵 | GET | /v3/realtest/heartbeat/check | 健康检查 | 0 | [HeartBeatController](../../app处理服务（real-test）/07-开放接口文档/基础设施/HeartBeatController.md) |
| 🔵 | GET | /v3/realtest/report/steps | app报告步骤 | 1 | [ReportController](../../app处理服务（real-test）/07-开放接口文档/报告与导出/ReportController.md) |
| 🔵 | POST | /v3/realtest/report/script_list | 问题分析列表 | 1 | [ProblemAnalysisReportController](../../app处理服务（real-test）/07-开放接口文档/性能与问题分析/ProblemAnalysisReportController.md) |
| 🔵 | POST | /v3/realtest/report/refresh_report_input_param | 刷新执行概要 | 1 | 同上 |
| 🔵 | GET | /v3/realtest/report/error_step_report_detail_list | 错误步骤详情 | 1 | 同上 |
| 🔵 | GET | /v3/realtest/report/device_list/{param1} | 问题分析设备列表 | 1 | 同上 |
| 🔵 | GET | /v3/realtest/template/copy | 复制app任务 | 1 | [TemplateController](../../app处理服务（real-test）/07-开放接口文档/报告与导出/TemplateController.md) |
| 🔵 | GET | /v3/realtest/plan/excel | 测试计划报告导出 | 1 | — |

### V3→V1 转换（passThroughType=0，V3 URL 匹配后转为 V1 请求）

| 转发 | method | V3 URL | → V1 action | → V1 op | 说明 | needLogin | 对应文档 |
|---|---|---|---|---|---|---|---|
| 🟡 | GET | /v3/realtest/task/share | app | Task.share | 分享报告生成skey | 1 | [TaskController](../../app处理服务（real-test）/07-开放接口文档/测试计划与任务/TaskController.md) |
| 🟡 | PUT | /v3/realtest/task/stop_task | app | Task.cancel | 取消某设备子任务 | 1 | 同上 |

---

## 2. web-pc处理服务（real-web）

> 模块 `realweb` / `realpc` / `common` → `http://testin-aio-real-web:8080`，接口文档索引：[real-web 接口索引](../../web-pc处理服务/07-开放接口文档/00-模块索引.md)

### V1 原生 — mkey=realweb

| 转发 | action | op | 说明 | needLogin | 对应文档 |
|---|---|---|---|---|---|
| 🟢 | task | Task.create | 创建web测试任务 | 1 | [service-task-Task](../../web-pc处理服务/07-开放接口文档/其他ApiServlet/service-task-Task.md) |
| 🟢 | task | Task.cancel / batchCancel | 取消/批量取消 | 1 | 同上 |
| 🟢 | task | Task.detail | 任务详情 | 1 | 同上 |
| 🟢 | task | Task.modify | 更新任务 | 1 | 同上 |
| 🟢 | task | Task.repeatTest | 补测 | 1 | 同上 |
| 🟢 | task | Task.scriptRunInfos / browserRunInfos | 执行详情 | 1 | 同上 |
| 🟢 | task | Task.sendEmail | 重发邮件 | 1 | 同上 |
| 🟢 | task | Task.modifyErrorMsg | 修改错误定位 | 1 | 同上 |
| 🟢 | task | Task.runInfoConditions | 执行概况查询条件 | 1 | [service-task-TestProcess](../../web-pc处理服务/07-开放接口文档/其他ApiServlet/service-task-TestProcess.md) |
| 🟢 | report | Report.list / scriptSteps / stepdetail / taskSummary / testProcess / scriptCheckInfos / scriptRunInfoSummary | 报告查询 | 1 | [ReportController](../../web-pc处理服务/07-开放接口文档/报告/ReportController.md) / [service-report-Report](../../web-pc处理服务/07-开放接口文档/其他ApiServlet/service-report-Report.md) |
| 🟢 | report | Report.getBrowserInfo / getClientInfo | 设备信息 | 1 | 同上 |
| 🟢 | report | Report.netReqDetail / stepInternetInfo / performanceCondition | 网络性能 | 1 | 同上 |
| 🟢 | report | Report.modifyResultCategory | 修改结果分类 | 1 | 同上 |
| 🟢 | report | Excel.reportExcel | Excel导出 | 1 | [service-report-Excel](../../web-pc处理服务/07-开放接口文档/其他ApiServlet/service-report-Excel.md) |
| 🟢 | report | Pdf.parse | PDF导出 | 1 | [service-report-Pdf](../../web-pc处理服务/07-开放接口文档/其他ApiServlet/service-report-Pdf.md) |
| 🟢 | dataSource | ParamDataSource.getDefaultDataSourceParam / getParamTableData | 数据源分配 | 0 | [service-dataSource-ParamDataSource](../../web-pc处理服务/07-开放接口文档/其他ApiServlet/service-dataSource-ParamDataSource.md) |

### V1 原生 — mkey=realpc

| 转发 | action | op | 说明 | needLogin | 对应文档 |
|---|---|---|---|---|---|
| 🟢 | task | Task.create / cancel / batchCancel / detail / modify / repeatTest | PC任务CRUD | 1 | 同上 Task |
| 🟢 | task | Task.clientRunInfos / scriptRunInfos / runInfoConditions | PC执行详情 | 1 | 同上 |
| 🟢 | report | Report.*（同 web 的报告 op） | PC报告查询 | 1 | 同上 Report |

### V1 原生 — mkey=common

| 转发 | action | op | 说明 | needLogin |
|---|---|---|---|---|
| 🟢 | common | System.currentTimeMillis | 获取当前时间 | 1 |

### V3→V1 转换 — mkey=device（V3 URL 转 V1，经 realweb 入口）

| 转发 | method | V3 URL | → V1 action | → V1 op | 说明 | needLogin | 对应文档 |
|---|---|---|---|---|---|---|---|
| 🟡 | GET | /v3/realweb/desktops | client | Client.disList | 桌面设备列表 | 1 | — |
| 🟡 | GET | /v3/realweb/devices | device | Device.list | 浏览器设备列表 | 1 | — |
| 🟡 | GET | /v3/realweb/webs | pc | Pc.list | 浏览器列表 | 1 | — |

### V3 透传 — mkey=realweb

| 转发 | method | URL | 说明 | needLogin | 对应文档 |
|---|---|---|---|---|---|
| 🔵 | GET | /v3/realweb/heartbeat/check | 健康检查 | 0 | [HeartBeatController](../../web-pc处理服务/07-开放接口文档/基础设施与统计/HeartBeatController.md) |
| 🔵 | POST | /v3/realweb/report/script_list | 问题分析列表 | 1 | [ProblemAnalysisReportController](../../web-pc处理服务/07-开放接口文档/报告/ProblemAnalysisReportController.md) |
| 🔵 | GET | /v3/realweb/report/steps | 报告步骤信息 | 1 | 同上 |
| 🔵 | GET | /v3/realweb/report/error_step_report_detail_list | 错误步骤详情 | 1 | 同上 |
| 🔵 | GET | /v3/realweb/report/device_list/{param1} | 设备列表 | 1 | 同上 |
| 🔵 | POST | /v3/realweb/report/refresh_report_input_param | 刷新执行概要 | 1 | 同上 |
| 🔵 | GET | /v3/realweb/template/copy | 复制web/pc任务 | 1 | — |
| 🔵 | GET | /v3/realweb/plan/excel | 测试计划报告导出 | 1 | — |

---

## 3. 任务管理服务（real-task）

> 模块 `real_task` → `http://testin-aio-real-task:8080`，接口文档索引：[real-task 接口索引](../../任务管理服务/07-开放接口文档/00-模块索引.md)

> 全部为 🔵 V3 透传（passThroughType=1），无 V1 原生或 V3→V1 转换。

| 转发 | method | URL | 说明 | needLogin | 对应文档 |
|---|---|---|---|---|---|
| 🔵 | POST | /v3/real_task/task_execute_records | 分页获取执行记录 | 1 | [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md) |
| 🔵 | GET | /v3/real_task/task_execute_records | 通过taskId获取信息 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/task_execute_records/execute | 触发任务模板执行 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/task_execute_report | 获取报告执行列表 | 1 | [TaskReportSummaryController](../../任务管理服务/07-开放接口文档/任务报告/TaskReportSummaryController.md) |
| 🔵 | GET | /v3/real_task/task_execute_record/report | 获取报告统计信息 | 1 | 同上 |
| 🔵 | GET | /v3/real_task/task_execute_record_standard_detail/list | 查询快照标准信息 | 1 | [TaskExecuteRecordStandardDetailController](../../任务管理服务/07-开放接口文档/任务报告/TaskExecuteRecordStandardDetailController.md) |
| 🔵 | GET | /v3/real_task/task_template | 查询模板列表 | 1 | [TaskTemplateController](../../任务管理服务/07-开放接口文档/任务模板管理/TaskTemplateController.md) |
| 🔵 | POST | /v3/real_task/task_template | 新增模板 | 1 | 同上 |
| 🔵 | GET | /v3/real_task/task_template/{param1} | 获取模板详情 | 1 | 同上 |
| 🔵 | PUT | /v3/real_task/task_template/{param1} | 更新模板 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/task_template/batch_delete | 批量删除模板 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/task_template/batch_delete_case | 批量删除用例 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/task_template/batch_delete_device | 批量删除设备 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/task_template/batch_pause | 批量暂停 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/task_template/batch_resume | 批量恢复 | 1 | 同上 |
| 🔵 | GET | /v3/real_task/task_template/cases | 查询用例列表 | 1 | 同上 |
| 🔵 | GET | /v3/real_task/task_template/devices | 查询设备列表 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/task_template/copy | 批量复制模板 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/cancel | 取消任务 | 1 | [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md) |
| 🔵 | POST | /v3/real_task/pause | 用例暂停 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/resume | 用例恢复 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/delete | 删除执行记录 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/case/task_execute_record | 查询用例执行记录 | 1 | [TaskExecuteRecordCaseController](../../任务管理服务/07-开放接口文档/用例报告/TaskExecuteRecordCaseController.md) |
| 🔵 | POST | /v3/real_task/case/list_report_case | 查询用例步骤执行记录 | 1 | 同上 |
| 🔵 | GET | /v3/real_task/case/report_case_detail/{param1} | 查询执行记录报告 | 1 | 同上 |
| 🔵 | PUT | /v3/real_task/case/modify_report_case/{param1} | 修改用例测试结果 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/case/modify_report_case_batch | 批量更新错误类型 | 1 | 同上 |
| 🔵 | DELETE | /v3/real_task/case/{param1} | 删除用例执行记录 | 1 | 同上 |
| 🔵 | GET | /v3/real_task/case/execute_case_statistic | 查询用例执行统计 | 1 | 同上 |
| 🔵 | GET | /v3/real_task/case_report/detail | 查询报告信息 | 1 | [TaskExecuteRecordCaseReportController](../../任务管理服务/07-开放接口文档/用例报告/TaskExecuteRecordCaseReportController.md) |
| 🔵 | GET | /v3/real_task/case_report/export_excel | 触发Excel导出 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/case_report/send_email | 发送报告邮件 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/case_report/share | 生成分享链接 | 1 | 同上 |
| 🔵 | POST | /v3/real_task/quick_debug/execute | 发起快速调试 | 1 | — |
| 🔵 | GET | /v3/real_task/quick_debug/{param1}/progress | 查询调试进度 | 1 | — |
| 🔵 | GET | /v3/real_task/quick_debug/{param1}/result | 获取调试结果 | 1 | — |
| 🔵 | POST | /v3/real_task/report/adopted | 采纳AI建议 | 1 | — |
| 🔵 | POST | /v3/real_task/report/auto_analysis | AI自动分析 | 1 | — |
| 🔵 | POST | /v3/real_task/report_steps | 报告左侧步骤树 | 1 | — |
| 🔵 | POST | /v3/real_task/report_step_detail | 报告步骤详情 | 1 | — |
| 🔵 | GET | /v3/real_task/summary/logs | 查询执行步骤日志 | 1 | — |
| 🔵 | GET | /v3/real_task/summary/report_performance | 查询性能数据 | 1 | — |
| 🔵 | GET | /v3/real_task/script_problem | 脚本问题分布 | 1 | — |

---

## 4. 任务调度服务（real-scheduling）

> 模块 `RealScheduling` → `http://testin-aio-real-scheduling:8080`，接口文档索引：[real-scheduling 接口索引](../../任务调度服务（real-scheduling）/07-开放接口文档/00-模块索引.md)

### V1 原生

| 转发 | action | op | 说明 | needLogin | 对应文档 |
|---|---|---|---|---|---|
| 🟢 | scheduling | Task.execute | 任务立即执行 | 0 | [scheduling-Task](../../任务调度服务（real-scheduling）/07-开放接口文档/任务调度/scheduling-Task.md) |
| 🟢 | scheduling | Task.list | 任务列表 | 0 | 同上 |
| 🟢 | scheduling | Task.levelTaskList | 升级任务列表 | 0 | 同上 |
| 🟢 | scheduling | Task.modifyLevel | 修改优先级 | 0 | 同上 |
| 🟢 | scheduling | Task.queueDetails | 设备任务数 | 0 | 同上 |
| 🟢 | scheduling | AbnormalDevice.* | 异常设备处理 | 0 | [scheduling-AbnormalDevice](../../任务调度服务（real-scheduling）/07-开放接口文档/任务调度/scheduling-AbnormalDevice.md) |
| 🟢 | scheduling | Cross.* | 跨模块任务调度 | 0 | [cross-Task](../../任务调度服务（real-scheduling）/07-开放接口文档/任务调度/cross-Task.md) |

### V3 透传

| 转发 | method | URL | 说明 | needLogin | 对应文档 |
|---|---|---|---|---|---|
| 🔵 | POST | /v3/RealScheduling/task/pauseTask | 暂停任务下发 | 1 | [DeviceTaskController](../../任务调度服务（real-scheduling）/07-开放接口文档/任务调度/DeviceTaskController.md) |
| 🔵 | POST | /v3/RealScheduling/task/resumeTask | 恢复任务下发 | 1 | 同上 |
| 🔵 | GET | /v3/RealScheduling/heartbeat/check | 健康检查 | 0 | [HeartBeatController](../../任务调度服务（real-scheduling）/07-开放接口文档/基础设施/HeartBeatController.md) |

---

## 5. 设备控制中心（real-controlcenter）

> 模块 `controlcenter` / `device` / `UcomDeivce` / `device_monitor` → `http://testin-aio-real-controlcenter:8080`，接口文档索引：[real-controlcenter 接口索引](../../设备控制中心（real-controlcenter）/07-开放接口文档/00-模块索引.md)

### V1 原生 — controlcenter（🟢，passThroughType=0，short action）

| 转发 | action | op 分组 | 说明 | needLogin | 对应文档 |
|---|---|---|---|---|---|
| 🟢 | control | Device.* | 设备占用/释放/截图/重启/串口/文件传输 | 0~1 | [control-Device](../../设备控制中心（real-controlcenter）/07-开放接口文档/设备管理/Device-control.md) |
| 🟢 | control | Task.command / Task.stop | 任务指令/停止 | 1 | [control-Task](../../设备控制中心（real-controlcenter）/07-开放接口文档/任务与预约/Task-control.md) |
| 🟢 | control | Command.response | 指令处理结果 | 1 | [Command](../../设备控制中心（real-controlcenter）/07-开放接口文档/客户端与升级/Command.md) |
| 🟢 | control | Pc.internalRestart | 重启上位机（转 Pc.restart） | 0 | [control-Pc](../../设备控制中心（real-controlcenter）/07-开放接口文档/PC与浏览器/Pc-control.md) |
| 🟢 | control | TaskGroup.executeTask | 任务组立即执行 | 0 | [TaskGroup](../../设备控制中心（real-controlcenter）/07-开放接口文档/任务与预约/TaskGroup.md) |
| 🟢 | device | Device.* | 设备查询/证书/机型/使用记录 | 0~1 | [Device](../../设备控制中心（real-controlcenter）/07-开放接口文档/设备管理/Device.md) |
| 🟢 | device | Certificate.* | iOS证书 | 1 | [Certificate](../../设备控制中心（real-controlcenter）/07-开放接口文档/基础设施/Certificate.md) |
| 🟢 | device | DeviceAssets.* | 设备资产 | 0 | [DeviceAssets](../../设备控制中心（real-controlcenter）/07-开放接口文档/设备管理/DeviceAssets.md) |
| 🟢 | device | DeviceStatistics.* | 设备统计 | 1 | [DeviceStatistics](../../设备控制中心（real-controlcenter）/07-开放接口文档/设备管理/DeviceStatistics.md) |
| 🟢 | device | UpgradeLog.* | 升级记录 | 1 | [UpgradeLog](../../设备控制中心（real-controlcenter）/07-开放接口文档/客户端与升级/UpgradeLog.md) |
| 🟢 | deviceGroup | DeviceGroup.* | 设备组CRUD | 1 | [DeviceGroup](../../设备控制中心（real-controlcenter）/07-开放接口文档/设备管理/DeviceGroup-deviceGroup.md) |
| 🟢 | pc | Pc.* | PC浏览器查询 | 1 | [Pc](../../设备控制中心（real-controlcenter）/07-开放接口文档/PC与浏览器/Pc-pc.md) |
| 🟢 | appointment | Appointment.* | 预约记录 | 1 | [Appointment](../../设备控制中心（real-controlcenter）/07-开放接口文档/任务与预约/Appointment.md) |
| 🟢 | cabinet | Cabinet.* | 机柜信息 | 0 | [Cabinet](../../设备控制中心（real-controlcenter）/07-开放接口文档/任务与预约/Cabinet.md) |
| 🟢 | db | Connection.testTry / Sql.execute | 数据库测试/SQL执行 | 0~1 | [Sql](../../设备控制中心（real-controlcenter）/07-开放接口文档/基础设施/Sql.md) |
| 🟢 | monitor | DeviceMonitor.* | 设备监控CRUD | 1 | [DeviceMonitor](../../设备控制中心（real-controlcenter）/07-开放接口文档/设备管理/DeviceMonitor.md) |
| 🟢 | monitor | DeviceAlarmLog.* | 报警记录 | 0~1 | [DeviceAlarmLog](../../设备控制中心（real-controlcenter）/07-开放接口文档/设备管理/DeviceAlarmLog.md) |

### V3 透传 — controlcenter

| 转发 | method | URL | 说明 | needLogin | 对应文档 |
|---|---|---|---|---|---|
| 🔵 | * | /v3/controlcenter/device/* | app设备管理（列表/详情/截图/重启/关机/资产） | 1 | [DeviceController](../../设备控制中心（real-controlcenter）/07-开放接口文档/设备管理/DeviceController.md) |
| 🔵 | * | /v3/controlcenter/web/* | web设备管理（统计/在线） | 1 | [WebController](../../设备控制中心（real-controlcenter）/07-开放接口文档/PC与浏览器/WebController.md) |
| 🔵 | * | /v3/controlcenter/sync_time/* | 时间同步配置 | 1 | [SyncTimeConfigController](../../设备控制中心（real-controlcenter）/07-开放接口文档/PC与浏览器/SyncTimeConfigController.md) |
| 🔵 | GET | /v3/controlcenter/heartbeat/check | 健康检查 | 0 | [HeartBeatController](../../设备控制中心（real-controlcenter）/07-开放接口文档/基础设施/HeartBeatController.md) |

### V3→V1 转换 — controlcenter（V3 URL 匹配后转 V1）

| 转发 | method | V3 URL | → V1 action | → V1 op | 说明 | needLogin |
|---|---|---|---|---|---|---|
| 🟡 | GET | /v3/controlcenter/device/device_assets_logs | device | DeviceAssets.logs | 设备资产日志查询 | 1 |
| 🟡 | POST | /v3/controlcenter/device/device_assets_logs | device | DeviceAssets.add | 设备资产添加 | 1 |
| 🟡 | POST | /v3/controlcenter/sql/execute | db | NewSql.execute | SQL执行 | 0 |

### V3 透传 — UcomDeivce（上位机上报，needLogin=0）

| 转发 | method | URL | 说明 | 对应文档 |
|---|---|---|---|---|
| 🔵 | POST | /v3/UcomDeivce/device/report | 手机设备上报 | [DeviceController-UcomDevice](../../设备控制中心（real-controlcenter）/07-开放接口文档/设备管理/DeviceController-UcomDevice.md) |
| 🔵 | POST | /v3/UcomDeivce/clientInfo/report | 桌面设备上报 | [ClientController](../../设备控制中心（real-controlcenter）/07-开放接口文档/客户端与升级/ClientController.md) |
| 🔵 | POST | /v3/UcomDeivce/browser/report | 浏览器上报 | [BrowserController](../../设备控制中心（real-controlcenter）/07-开放接口文档/PC与浏览器/BrowserController.md) |
| 🔵 | POST | /v3/UcomDeivce/harmonyPc/report | 鸿蒙PC上报 | [HarmonyPcController](../../设备控制中心（real-controlcenter）/07-开放接口文档/PC与浏览器/HarmonyPcController.md) |
| 🔵 | POST | /v3/UcomDeivce/task/complete | 任务完成上报 | [TaskController](../../设备控制中心（real-controlcenter）/07-开放接口文档/任务与预约/TaskController.md) |
| 🔵 | POST | /v3/UcomDeivce/task/precomplete | 预完成上报 | 同上 |
| 🔵 | POST | /v3/UcomDeivce/task/processReport | 过程上报 | 同上 |
| 🔵 | POST | /v3/UcomDeivce/task/resultReport | 结果上报 | 同上 |
| 🔵 | POST | /v3/UcomDeivce/task/videoReport | 视频上报 | 同上 |
| 🔵 | POST | /v3/UcomDeivce/device/occupy | 设备占用 | [DeviceController-UcomDevice](../../设备控制中心（real-controlcenter）/07-开放接口文档/设备管理/DeviceController-UcomDevice.md) |
| 🔵 | POST | /v3/UcomDeivce/device/release | 设备释放 | 同上 |

---

## 6. 平台基础功能服务（testin-core）

> 模块 `core` / `usermanager` / `user` / `notice` / `project` / `test_plan` / `test_case` / `task` / `realportal` / `defect` / `logfile` / `log` → `http://testin-aio-testin-core:8080`，接口文档索引：[testin-core 接口索引](../../平台基础功能服务/07-开放接口文档/00-模块索引.md)

### V1 原生 — user / usermanager（用户与认证）

| 转发 | action | op | 说明 | 对应文档 |
|---|---|---|---|---|
| 🟢 | user | Login.login / Login.logout | 登录/登出 | [service-Login](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Login.md) |
| 🟢 | user | Online.getUserOnline / getOnlineNums | 在线状态 | [service-Online](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Online.md) |
| 🟢 | user | User.getToken / getUser / getAllUserInfo / getProjectUserList | 用户信息 | [service-User](../../平台基础功能服务/07-开放接口文档/用户与认证/service-User.md) |
| 🟢 | user | Enterprise.* | 企业管理 | [service-Enterprise](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Enterprise.md) |
| 🟢 | user | Project.* | 项目管理 | [service-Project](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Project.md) |
| 🟢 | user | Role.roleByEidAndProjectId | 角色查询 | [service-Role](../../平台基础功能服务/07-开放接口文档/用户与认证/service-Role.md) |
| 🟢 | user | SystemCfg.getSystemConfig / sso | 系统配置/SSO | [service-SystemCfg](../../平台基础功能服务/07-开放接口文档/用户与认证/service-SystemCfg.md) |
| 🟢 | user | SystemParam.* | 系统参数 | [service-SystemParam](../../平台基础功能服务/07-开放接口文档/用户与认证/service-SystemParam.md) |

### V3 透传 + V3→V1 转换 — user（用户接口，两种模式并存）

| 转发 | method | URL | → V1 action | → V1 op | 说明 | 对应文档 |
|---|---|---|---|---|---|---|
| 🔵 | POST | /v3/core/users/login | — | — | 登录 | [UserController](../../平台基础功能服务/07-开放接口文档/用户与认证/UserController.md) |
| 🟡 | POST | /v3/core/users/login | user | Login.login | 登录（V3→V1，needLogin=0） | 同上 |
| 🔵 | POST | /v3/core/users/logout | — | — | 登出 | 同上 |
| 🟡 | POST | /v3/core/users/logout | user | Login.logout | 登出（V3→V1，needLogin=0） | 同上 |
| 🔵 | GET | /v3/core/user_info | — | — | 获取sid在线信息 | 同上 |
| 🟡 | GET | /v3/core/user_info | user | Online.getUserOnline | 获取在线信息（V3→V1） | 同上 |
| 🔵 | GET | /v3/core/users/online_nums | — | — | 在线人数 | 同上 |
| 🟡 | GET | /v3/core/users/online_nums | user | Online.getOnlineNums | 在线人数（V3→V1） | 同上 |
| 🔵 | PUT | /v3/core/users/projects/{param1} | — | — | 切换项目 | 同上 |
| 🟡 | PUT | /v3/core/users/projects/{param1} | user | Project.modifyAccessTime | 切换项目（V3→V1） | 同上 |
| 🔵 | * | /v3/core/enterprise/* | — | — | 企业管理 | [EnterpriseController](../../平台基础功能服务/07-开放接口文档/用户与认证/EnterpriseController.md) |
| 🔵 | * | /v3/core/project/* | — | — | 项目管理 | [ProjectController](../../平台基础功能服务/07-开放接口文档/用户与认证/ProjectController.md) |

> **说明**：user 模块的登录/登出/在线等接口有两条数据库记录（不同角色），一条 passThroughType=1 直接透传，另一条 passThroughType=0 转换为 V1。运行时根据调用方角色匹配到哪条记录决定转发路径。

### V3 透传 — core（管理后台类接口）

| 转发 | method | URL | 说明 | 对应文档 |
|---|---|---|---|---|
| 🔵 | POST | /v3/core/action/records | 操作记录查询 | — |
| 🔵 | GET | /v3/core/bi/project_statistic | 项目统计 | — |
| 🔵 | GET | /v3/core/bi/user_statistic | 用户统计 | — |
| 🔵 | * | /v3/core/email_template/* | 邮件模板管理 | — |
| 🔵 | * | /v3/core/quartz_dir/* | 定时任务目录管理 | — |
| 🔵 | * | /v3/core/server/disk/* | 磁盘监控 | — |

### V3→V1 转换 — core

| 转发 | method | V3 URL | → V1 action | → V1 op | 说明 |
|---|---|---|---|---|---|
| 🟡 | GET | /v3/core/action/record | common | ActionLogController.getActionLog | 操作记录查询 |

### V3 透传 — notice（通知服务）

| 转发 | method | URL | 说明 | 对应文档 |
|---|---|---|---|---|
| 🔵 | POST | /v3/notice/channel/add_channel | 添加消息渠道 | [NoticeChannelCfgController](../../平台基础功能服务/07-开放接口文档/通知与消息/NoticeChannelCfgController.md) |
| 🔵 | GET | /v3/notice/channel/list | 查询消息渠道 | 同上 |
| 🔵 | DELETE | /v3/notice/channel/{param1} | 删除消息渠道 | 同上 |
| 🔵 | PUT | /v3/notice/channel/{param1} | 修改消息渠道 | 同上 |
| 🔵 | POST | /v3/notice/notice/add_event | 新增消息事件 | [NoticeEventController](../../平台基础功能服务/07-开放接口文档/通知与消息/NoticeEventController.md) |
| 🔵 | GET | /v3/notice/notice/list | 查询消息事件 | 同上 |
| 🔵 | GET | /v3/notice/log/list | 查询消息日志 | [NoticeLogController](../../平台基础功能服务/07-开放接口文档/通知与消息/NoticeLogController.md) |

### V1 原生 — notice（传统 action/op）

| 转发 | action | op | 说明 | 对应文档 |
|---|---|---|---|---|
| 🟢 | email | EmailTask.add / EmailTempletCfg.* / EmailCfg.testSendEmail | 邮件任务/模板/测试 | — |
| 🟢 | newChannel | Channel.* / MsgTask.add | 消息渠道管理 | — |
| 🟢 | qc | QcCfg.list | QC配置 | — |
| 🟢 | sms | SmsTemplet.list | 短信模板 | — |

### V3 透传 — project（项目管理）

| 转发 | method | URL | 说明 | 对应文档 |
|---|---|---|---|---|
| 🔵 | GET | /v3/core/projects/list | 项目列表 | — |
| 🔵 | POST | /v3/core/projects/operate_users | 操作用户 | — |
| 🔵 | GET | /v3/core/projects/users | 项目用户 | — |
| 🔵 | GET | /v3/core/projects/{param1} | 项目详情 | — |
| 🔵 | PUT | /v3/core/projects/{param1} | 更新项目 | — |

### V3→V1 转换 — project

| 转发 | method | V3 URL | → V1 action | → V1 op | 说明 |
|---|---|---|---|---|---|
| 🟡 | GET | /v3/core/projects | user | Project.getProjectList | 项目列表 |
| 🟡 | POST | /v3/core/projects | user | Project.addProject | 添加项目 |

### test_plan（测试计划）

全部 🔵 V3 透传（passThroughType=1） → 对应文档：[PlanInfoController](../../平台基础功能服务/07-开放接口文档/测试计划/PlanInfoController.md) 及相关

| 转发 | method | URL 前缀 | 说明 |
|---|---|---|---|
| 🔵 | */v3/test_plan/test_plans* | CRUD/execute/copy | 测试计划 CRUD 与执行触发 |
| 🔵 | */v3/test_plan/sub_plans* | CRUD | 子计划管理 |
| 🔵 | */v3/test_plan/plan_tasks* | CRUD/batch_delete | 任务模板关联 |
| 🔵 | */v3/test_plan/execute_records* | GET/DELETE/PUT | 执行记录查询/删除/更新 |
| 🔵 | */v3/test_plan/execute_record_tasks* | GET/reset/stop/callback | 执行记录任务树/重测/停止/回调 |

### test_case（测试用例）

全部 🔵 V3 透传（passThroughType=1） → 对应文档：[CaseInfoController](../../平台基础功能服务/07-开放接口文档/测试用例/CaseInfoController.md)

| 转发 | method | URL 前缀 | 说明 |
|---|---|---|---|
| 🔵 | */v3/test_case/case_info* | CRUD/copy/import/export | 用例 CRUD 与导入导出 |
| 🔵 | */v3/test_case/case_dir* | CRUD/move | 用例目录管理 |
| 🔵 | */v3/test_case/case_step* | CRUD/move | 用例步骤管理 |
| 🔵 | */v3/test_case/case_comment* | CRUD | 用例评论 |
| 🔵 | */v3/test_case/case_statistic* | GET | 用例统计 |

### task（任务管理 / Portal）

V1 原生：
| 转发 | action | op | 说明 | 对应文档 |
|---|---|---|---|---|
| 🟢 | portal | Task.* | Portal任务CRUD | [portal-Task](../../平台基础功能服务/07-开放接口文档/任务管理/portal-Task.md) |

V3 透传 + V3→V1：
| 转发 | method | URL | → V1 action | → V1 op | 说明 | 对应文档 |
|---|---|---|---|---|---|---|
| 🔵 | * | /v3/task/tasks | — | — | 任务CRUD（透传） | [TaskController](../../平台基础功能服务/07-开放接口文档/任务管理/TaskController.md) |
| 🟡 | GET | /v3/task/tasks | portal | Task.list | 任务列表（V3→V1） | 同上 |

### V1 原生 — realportal

| 转发 | action | op | 说明 | 对应文档 |
|---|---|---|---|---|
| 🟢 | portal | Task.refresh / delete / batchDelete / get / list | Portal任务管理 | [portal-Task](../../平台基础功能服务/07-开放接口文档/任务管理/portal-Task.md) |

### V3 透传 — defect / log / logfile / report

| mkey | 转发 | method | URL | 说明 |
|---|---|---|---|---|
| defect | 🔵 | * | /v3/defect/* | 缺陷管理 CRUD |
| log | 🔵 | * | /v3/log/logs/* | 操作日志查询 |
| logfile | 🟢 | real | LogFile.* | 日志文件查询（V1原生） |

---

## 7. 平台配置（real-cfg）

> 模块 `realcfg` / `env` → `http://testin-aio-real-cfg:8080`，接口文档索引：[real-cfg 接口索引](../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md)

### V1 原生 — mcfg 配置管理（内部接口，needLogin=0）

| 转发 | action | op | 说明 | 网关缓存 | 对应文档 |
|---|---|---|---|---|---|
| 🟢 | mcfg | AppCfg.get / list / add / maintain / delete | 应用配置CRUD | IAppService | [AppCfg](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/AppCfg.md) |
| 🟢 | mcfg | ModuleCfg.get / list | 模块配置 | IModuleService | [ModuleCfg](../../平台配置（real-cfg）/07-开放接口文档/用户与权限/ModuleCfg.md) |
| 🟢 | mcfg | ApiCfg.list / listByRole | API配置查询 | IApiService | [ApiCfg](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/ApiCfg.md) |
| 🟢 | mcfg | RoleCfg.get / list / add / maintain / delete | 角色CRUD | IApiService.refresh | [RoleCfg](../../平台配置（real-cfg）/07-开放接口文档/用户与权限/RoleCfg.md) |
| 🟢 | mcfg | RoleCfg.addApi / removeApi / maintainApi | 角色API关联 | IApiService.refresh | 同上 |

### V1 原生 — 业务配置

| 转发 | action | op 分组 | 说明 | 对应文档 |
|---|---|---|---|---|
| 🟢 | cfg | BizConfig.get | 业务配置 | [BizConfig](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/BizConfig.md) |
| 🟢 | cfg | Brand.list / Model.list | 品牌/机型 | [Brand](../../平台配置（real-cfg）/07-开放接口文档/项目与平台配置/Brand.md) / [Model](../../平台配置（real-cfg）/07-开放接口文档/项目与平台配置/Model.md) |
| 🟢 | cfg | EnvCfg.* | 环境管理 | [EnvCfg](../../平台配置（real-cfg）/07-开放接口文档/项目与平台配置/EnvCfg.md) |
| 🟢 | cfg | DbCfg.* | 数据库配置 | [DbCfg](../../平台配置（real-cfg）/07-开放接口文档/数据源与代码配置/DbCfg.md) |
| 🟢 | cfg | CANCfg.* / DAQCfg.* | CAN/DAQ配置 | [CANCfg](../../平台配置（real-cfg）/07-开放接口文档/数据源与代码配置/CANCfg.md) / [DAQCfg](../../平台配置（real-cfg）/07-开放接口文档/数据源与代码配置/DAQCfg.md) |
| 🟢 | cfg | PcAccount.* / PcCfg.* | 上位机账号/配置 | [PcAccount](../../平台配置（real-cfg）/07-开放接口文档/用户与权限/PcAccount.md) / [PcCfg](../../平台配置（real-cfg）/07-开放接口文档/用户与权限/PcCfg.md) |
| 🟢 | cfg | FunctionSwitch.* | 功能开关 | [FunctionSwitch](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/FunctionSwitch.md) |
| 🟢 | cfg | Timeout.* | 全局超时配置 | [Timeout](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/Timeout.md) |
| 🟢 | cfg | PreciseTestCfg.* | 精准测试配置 | — |
| 🟢 | cfg | ProjectGroup.my | 设备云信息 | [ProjectGroup](../../平台配置（real-cfg）/07-开放接口文档/项目与平台配置/ProjectGroup.md) |
| 🟢 | RealCfg | ServiceCfg.* | 角色权限 | [ServiceCfg](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/ServiceCfg.md) |
| 🟢 | cfg | DeviceCfg.* | 设备配置 | [DeviceCfg](../../平台配置（real-cfg）/07-开放接口文档/设备与网络配置/DeviceCfg.md) |
| 🟢 | cfg | DeviceGroup.* | 设备组配置 | [DeviceGroup](../../平台配置（real-cfg）/07-开放接口文档/设备与网络配置/DeviceGroup.md) |
| 🟢 | cfg | NetworkCfg.* | 网络配置 | [NetworkCfg](../../平台配置（real-cfg）/07-开放接口文档/设备与网络配置/NetworkCfg.md) |
| 🟢 | cfg | RaspiCfg.* | 树莓派配置 | [RaspiCfg](../../平台配置（real-cfg）/07-开放接口文档/设备与网络配置/RaspiCfg.md) |

### V3 透传

| 转发 | method | URL | 说明 | 对应文档 |
|---|---|---|---|---|
| 🔵 | GET | /v3/realcfg/envs | 环境列表 | [EnvController](../../平台配置（real-cfg）/07-开放接口文档/项目与平台配置/EnvController.md) |
| 🔵 | * | /v3/realcfg/error_cause_* | 错误原因/类型管理 | [ErrorCauseTypeController](../../平台配置（real-cfg）/07-开放接口文档/基础设施与问题管理/ErrorCauseTypeController.md) / [ErrorCauseOperateLogController](../../平台配置（real-cfg）/07-开放接口文档/基础设施与问题管理/ErrorCauseOperateLogController.md) |
| 🔵 | GET | /v3/realcfg/network | 弱网参数列表 | [NetworkController](../../平台配置（real-cfg）/07-开放接口文档/设备与网络配置/NetworkController.md) |
| 🔵 | * | /v3/realcfg/project/* | 项目高级设置 | [ProjectAdvancedConfigController](../../平台配置（real-cfg）/07-开放接口文档/项目与平台配置/ProjectAdvancedConfigController.md) |
| 🔵 | * | /v3/realcfg/swj/* | 上位机配置 | [UcomIdController](../../平台配置（real-cfg）/07-开放接口文档/基础设施与问题管理/UcomIdController.md) |
| 🔵 | GET | /v3/realcfg/ucomid | 新增上位机账号 | 同上 |
| 🔵 | GET | /v3/realcfg/heartbeat/check | 健康检查 | [HeartBeatController](../../平台配置（real-cfg）/07-开放接口文档/基础设施与问题管理/HeartBeatController.md) |

---

## 8. 脚本服务（filesystem）

> 模块 `script` / `apppackage` / `suite` / `file` → `http://testin-aio-filesystem:8080`，接口文档索引：[脚本服务 接口索引](../../脚本服务/07-开放接口文档/00-模块索引.md)

### V1 原生 — script（🟢，passThroughType=0，short action）

| 转发 | action | op 分组 | 说明 | 对应文档 |
|---|---|---|---|---|
| 🟢 | script | Script.* (~35 op) | 脚本CRUD/批量操作/历史版本/上传解析/复制/还原 | [Script](../../脚本服务/07-开放接口文档/脚本管理/Script.md) |
| 🟢 | script | Collections.* (~6 op) | 脚本目录树 | [Collections](../../脚本服务/07-开放接口文档/目录管理/Collections.md) |
| 🟢 | script | ScriptGroup.* (~7 op) | 脚本组管理 | [Script](../../脚本服务/07-开放接口文档/脚本管理/Script.md) |
| 🟢 | script | ScriptRecord.* (~3 op) | 脚本录制/保存/AI分析 | [ScriptRecord](../../脚本服务/07-开放接口文档/脚本管理/ScriptRecord.md) |
| 🟢 | script | ExportScript.* (~4 op) | 导出 | [ExportScript](../../脚本服务/07-开放接口文档/脚本管理/ExportScript.md) |
| 🟢 | script | ScriptList.* (~7 op) | 脚本资源查询 | [ExportScript](../../脚本服务/07-开放接口文档/脚本管理/ExportScript.md) |
| 🟢 | script | RecycleBin.* | 回收站 | [RecycleBin](../../脚本服务/07-开放接口文档/脚本管理/RecycleBin.md) |
| 🟢 | script | Parameter.* | 参数管理 | [Parameter](../../脚本服务/07-开放接口文档/参数与数据源/Parameter.md) |
| 🟢 | script | DataSource.list | 数据源列表 | [DataSource](../../脚本服务/07-开放接口文档/参数与数据源/DataSource.md) |
| 🟢 | script | Appcontrol.* | 控件管理 | [Appcontrol](../../脚本服务/07-开放接口文档/App与控件/Appcontrol.md) |
| 🟢 | script | CleanScript.* | 脚本清理 | [CleanScript](../../脚本服务/07-开放接口文档/脚本管理/CleanScript.md) |
| 🟢 | suite | Suite.* | 应用管理 | [Suite](../../脚本服务/07-开放接口文档/套件管理/Suite.md) |
| 🟢 | form | BoxStatus.* / FormResource.* | 表单标注 | [FormResource](../../脚本服务/07-开放接口文档/表单资源/FormResource.md) |
| 🟢 | tab | TabConfig.* | PC内嵌浏览器 | [TabConfig](../../脚本服务/07-开放接口文档/基础设施/TabConfig.md) |
| 🟢 | voice | DbcFile.* / VocContentDetail.* / VocLibrary.* | 语音/语料 | — |
| 🟢 | thirdparty | Script.* | 第三方脚本绑定 | [thirdparty-Script](../../脚本服务/07-开放接口文档/第三方集成/thirdparty-Script.md) |
| 🟢 | cicc | PreciseTest.* | 精准测试 | — |
| 🟢 | ai | Resources.* | AI资源 | [Resources](../../脚本服务/07-开放接口文档/AI资源标注/Resources.md) |
| 🟢 | scriptGroup | add / update / delete | 脚本组管理 | — |

### V3 透传 — script（🔵，passThroughType=1）

| 转发 | method | URL | 说明 | 对应文档 |
|---|---|---|---|---|
| 🔵 | * | /v3/script/scripts* | 脚本CRUD/导入/导出/复制 | [ScriptController](../../脚本服务/07-开放接口文档/脚本管理/ScriptController.md) |
| 🔵 | * | /v3/script/script_groups* | 脚本组管理 | [ScriptUpgradeController](../../脚本服务/07-开放接口文档/脚本管理/ScriptUpgradeController.md) |
| 🔵 | * | /v3/script/python_script* | Python脚本CRUD/调试/执行 | [pythonController](../../脚本服务/07-开放接口文档/Python脚本/pythonController.md) |
| 🔵 | * | /v3/script/mark/* | AI资源标注 | [ResourcesMarkController](../../脚本服务/07-开放接口文档/AI资源标注/ResourcesMarkController.md) |
| 🔵 | * | /v3/script/script_record/* | 录制管理 | [ScriptRecordController](../../脚本服务/07-开放接口文档/脚本管理/ScriptRecordController.md) |
| 🔵 | GET | /v3/script/location | 位置查询 | — |
| 🔵 | GET | /v3/script/heartbeat/check | 健康检查 | [HeartBeatController](../../脚本服务/07-开放接口文档/基础设施/HeartBeatController.md) |

### V3→V1 转换 — script（🟡，V3 URL → V1 action/op）

| 转发 | method | V3 URL | → V1 action | → V1 op | 说明 |
|---|---|---|---|---|---|
| 🟡 | POST | /v3/script/dirs | script | Collections.add | 添加目录 |
| 🟡 | GET | /v3/script/dirs | script | Collections.list | 目录列表 |
| 🟡 | DELETE | /v3/script/dirs/{param1} | script | Collections.maintain | 删除目录 |
| 🟡 | PUT | /v3/script/dirs/{param1} | script | Collections.maintain | 更新目录 |
| 🟡 | POST | /v3/script/dirs/{param1}/copy | script | Script.dirCopy | 复制目录 |
| 🟡 | GET | /v3/script/dirs/script_nums | script | Script.scriptNum | 脚本数量统计 |
| 🟡 | GET | /v3/script/scripts/datasources/export/{param1} | script | ExportScript.getResult | 导出结果查询 |
| 🟡 | GET | /v3/script/scripts/depend_relations/{param1} | script | Script.getParentOrChildScriptsByScriptNo | 脚本依赖关系 |
| 🟡 | POST | /v3/script/scripts/files/parse | script | Script.parse | 脚本文件解析 |
| 🟡 | POST | /v3/script/scripts/import | script | Script.importScript | 导入脚本 |
| 🟡 | POST | /v3/script/scripts/import_url | script | Script.importScriptUrl | URL导入脚本 |
| 🟡 | POST | /v3/script/scripts/pre_import | script | Script.suppImportScriptList | 预导入脚本列表 |
| 🟡 | POST | /v3/script/script_groups/copy | script | ScriptGroup.copy | 复制脚本组 |
| 🟡 | GET | /v3/script/script_nos/{param1} | script | Collections.scriptList | 脚本号列表 |

> **说明**：script 模块的 V3 URL 接口中，目录管理类（dirs）和部分脚本操作类为 passThroughType=0 + special_api_action/op 的 V3→V1 转换模式。脚本CRUD主接口（/scripts*）为 passThroughType=1 透传。

### V3 透传 — suite（套件管理）

| 转发 | method | URL | 说明 | 对应文档 |
|---|---|---|---|---|
| 🔵 | GET | /v3/script/suites/app_packages | App包查询 | — |

### V3→V1 转换 — suite

| 转发 | method | V3 URL | → V1 action | → V1 op | 说明 | 对应文档 |
|---|---|---|---|---|---|---|
| 🟡 | POST | /v3/script/suites | suite | Suite.add | 添加套件 | [SuiteController](../../脚本服务/07-开放接口文档/套件管理/SuiteController.md) |
| 🟡 | GET | /v3/script/suites | suite | Suite.list | 套件列表 | 同上 |
| 🟡 | GET | /v3/script/suites/{param1} | suite | Suite.get | 套件详情 | 同上 |
| 🟡 | PUT | /v3/script/suites/{param1} | suite | Suite.modify | 更新套件 | 同上 |
| 🟡 | DELETE | /v3/script/suites/{param1} | suite | Suite.delete | 删除套件 | 同上 |

### V1 原生 — apppackage（App包查询）

| 转发 | action | op | 说明 | 对应文档 |
|---|---|---|---|---|
| 🟢 | app | AppPackage.listPackageFile / listAppVersion / getPackageFile / completeSingInfo | 应用包查询/版本列表/签名信息 | [AppPackage](../../脚本服务/07-开放接口文档/App与控件/AppPackage.md) |
| 🟢 | app | App.addApp / certificate | 应用管理 | [App](../../脚本服务/07-开放接口文档/App与控件/App.md) |
| 🟢 | app | AppSubInfo.* | 子应用管理 | [AppSubInfo](../../脚本服务/07-开放接口文档/App与控件/AppSubInfo.md) |

### V1 原生 — file

| 转发 | action | op | 说明 | 对应文档 |
|---|---|---|---|---|
| 🟢 | file | FileApi.upload | 文件上传 | [FileApi](../../脚本服务/07-开放接口文档/基础设施/FileApi.md) |
| 🟢 | file | FileApi.listFileByIds | 按ID查文件 | 同上 |

---

## 9. 文件管理服务（fileupload）

> 模块 `fs` / `file_system` → `http://testin-aio-fileupload:8080/openapi`，接口文档索引：[文件管理服务 接口索引](../../文件管理服务/07-开放接口文档/00-模块索引.md)

### V1 原生（action/op 模式）

| 转发 | action | op | 说明 | 对应文档 |
|---|---|---|---|---|
| 🟢 | fs | File.upload / internalUpload / multipartUpload / merge | 文件上传/分片/合并 | [service-File](../../文件管理服务/07-开放接口文档/ApiServlet/service-File.md) |
| 🟢 | fs | App.parse | App解析 | [service-App](../../文件管理服务/07-开放接口文档/ApiServlet/service-App.md) |
| 🟢 | fs | Script.checkScript | 脚本检测 | [service-ScriptMain](../../文件管理服务/07-开放接口文档/ApiServlet/service-ScriptMain.md) |
| 🟢 | fs | TestResult.parse | 结果解析 | [service-TestResult](../../文件管理服务/07-开放接口文档/ApiServlet/service-TestResult.md) |

> `File.internalUpload` 设置 `special_api_op=File.upload`（V1内部重定向，非V3→V1转换）。

### V3 透传（passThroughType=1）

| 转发 | method | URL | 说明 | 对应文档 |
|---|---|---|---|---|
| 🔵 | POST | /openapi/v3/fs/app/upload | app上传保存 | [AppV3Controller](../../文件管理服务/07-开放接口文档/文件上传/AppV3Controller.md) |
| 🔵 | POST | /openapi/v3/fs/files/upload | 表单/分片上传 | [FileUploadV3Controller](../../文件管理服务/07-开放接口文档/文件上传/FileUploadV3Controller.md) |
| 🔵 | * | /openapi/v3/fs/app/* | App管理（版本/签名/包查询） | [AppController](../../文件管理服务/07-开放接口文档/文件上传/AppController.md) |
| 🔵 | * | /openapi/v3/fs/file/* | 文件操作（上传/下载/删除/批量） | [FileUploadController](../../文件管理服务/07-开放接口文档/文件上传/FileUploadController.md) |
| 🔵 | * | /openapi/v3/fs/script/* | 脚本管理 | [ScriptV3Controller](../../文件管理服务/07-开放接口文档/脚本管理/ScriptV3Controller.md) |
| 🔵 | * | /openapi/v3/fs/url/* | URL管理 | [UrlController](../../文件管理服务/07-开放接口文档/文件上传/UrlController.md) |
| 🔵 | * | /openapi/v3/fs/package/* | 包管理 | [PackageController](../../文件管理服务/07-开放接口文档/文件上传/PackageController.md) |
| 🔵 | * | /openapi/v3/fs/query/* | 文件查询 | [QueryController](../../文件管理服务/07-开放接口文档/查询服务/QueryController.md) |
| 🔵 | GET | /openapi/v3/fs/heartbeat/check | 健康检查 | [HeartBeatController](../../文件管理服务/07-开放接口文档/内部工具/HeartBeatController.md) |

---

## 10. 数据源（datasource-manage）

> 模块 `datasource` / `datasource-manage` → `http://testin-aio-datasource-manage:8080/openapi`，接口文档索引：[数据源 接口索引](../../数据源/07-开放接口文档/00-模块索引.md)

### V1 原生（action/op 模式）

| 转发 | action | op 分组 | 说明 | 对应文档 |
|---|---|---|---|---|
| 🟢 | source | SourceConfigCtrl.* (~19 op) | 数据源CRUD/导入导出/绑定/复制/移动 | [service-SourceConfigCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-SourceConfigCtrl.md) |
| 🟢 | source | DataTableCtrl.* (~7 op) | 表格数据操作 | [service-DataTableCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-DataTableCtrl.md) |
| 🟢 | source | ColConfigCtrl.* (~3 op) | 列配置 | [service-ColConfigCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-ColConfigCtrl.md) |
| 🟢 | source | SqlCtrl.* (~3 op) | SQL管理 | [service-SqlCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-SqlCtrl.md) |
| 🟢 | source | TagInfoCtrl.* (~3 op) | 标签管理 | [service-TagInfoCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-TagInfoCtrl.md) |
| 🟢 | source | SelectCtrl.* (~4 op) | 全局搜索 | [service-SelectCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-SelectCtrl.md) |
| 🟢 | source | MigrateCtrl.fillGlobalInfo | 全局表迁移 | [service-MigrateCtrl](../../数据源/07-开放接口文档/其他ApiServlet/service-MigrateCtrl.md) |

### V3 透传（passThroughType=1）

| 转发 | method | URL | 说明 | 对应文档 |
|---|---|---|---|---|
| 🔵 | * | /v3/datasource-manage/datasources* | 数据源列表 | [DataSourceController](../../数据源/07-开放接口文档/数据源管理/DataSourceController.md) |
| 🔵 | * | /v3/datasource-manage/normal/* | 存储数据源CRUD | [NormalSourceController](../../数据源/07-开放接口文档/数据源管理/NormalSourceController.md) |
| 🔵 | * | /v3/datasource-manage/test_case/* | 用例数据源CRUD | [CaseSourceController](../../数据源/07-开放接口文档/数据源管理/CaseSourceController.md) |
| 🔵 | * | /v3/datasource-manage/project/global_variables* | 项目全局变量 | [ProjectGlobalVariablesController](../../数据源/07-开放接口文档/数据源管理/ProjectGlobalVariablesController.md) |
| 🔵 | POST | /openapi/v3/datasource-manage/getScriptParamDataNew | 获取表格数据 | [DataSourceController](../../数据源/07-开放接口文档/数据源管理/DataSourceController.md) |
| 🔵 | POST | /openapi/v3/datasource-manage/source_configs | 数据源列表 | 同上 |
| 🔵 | * | /v3/datasource-manage/tag_infos* | 标签管理 | [TagInfoController](../../数据源/07-开放接口文档/数据源管理/TagInfoController.md) |
| 🔵 | GET | /v3/datasource-manage/heartbeat/check | 健康检查 | [HeartBeatController](../../数据源/07-开放接口文档/数据源管理/HeartBeatController.md) |

---

## 未归属知识库的服务（仅记录，无文档链接）

| mkey | 目标服务 | API数 | 主要转发模式 | 说明 |
|---|---|---|---|---|
| admin | 管理中心 | 50 | 🔵 透传 + 🟡 V3→V1 混合 | 运维后台：设备资产管理/天统计/用户管理/日志查询 |
| analysis | 分析服务 | — | — | testin-aio-real-analysis:8080 |
| devops | 环境运维 | 49 | 🟢 V1原生 | testin-aio-devops:8080 |
| testin-plan | 测试计划(独立) | 36 | 🟢 V1原生 | testin-aio-test-plan:8080/openapi |
| testin-third | 第三方缺陷 | 21 | 🟢 V1原生 | testin-aio-testin-third:8080/openapi |
| prodatatrans | 数据传输 | 32 | 🟢 V1原生 | testin-aio-datatrans:8080 |
| realcross | 跨端 | 10 | 🟢 V1原生 | testin-aio-real-cross:8080 |
| remote_report | 真机调试 | 25 | 🟢 V1原生 | testin-aio-assistant-web:8080/openapi |
| statis | 统计 | — | — | testin-aio-statis:8080 |
| TestManageAdapt | 测管提测 | 1 | 🟢 V1原生 | testin-aio-testmanageadapt:8080/openapi |
| opsapi | 运维WebShell | 1 | 🟢 V1原生 | testin-aio-serverops:8080 |
| report | 报表 | 2 | 🔵 V3透传 | testin-aio-testin-core:8080 |
| logfile | 日志服务 | 8 | 🟢 V1原生 | testin-aio-testin-core:8080 |

---

## 统计

| 分类 | 模块数 | API总数（去重约） | V1原生 | V3透传 | V3→V1转换 |
|---|---|---|---|---|---|
| 已归属知识库 | 30 | ~1200 | ~520 | ~590 | ~90 |
| 未归属（外部/运维/独立部署） | 13 | ~160 | ~140 | ~15 | ~5 |
| **合计** | **43** | **~1365** | **~660** | **~605** | **~95** |

### 转发模式分布（全库 db_mcfg 统计）

| 转发模式 | pass_through_type | 判定 | API 数 | 占比 |
|---|---|---|---|---|
| 🟢 V1 原生 | 0 | api_action 为短格式（如 `app.Task.add`） | ~660 | 48% |
| 🔵 V3 透传 | **1** | api_action 为 URL 格式 | 487 | 36% |
| 🟡 V3→V1 转换 | **0** | URL 格式 + special_api_action/op 非空 | ~218 | 16% |

> **说明**：passThroughType=0 共 875 条记录，其中 ~660 条为纯 V1 原生（short action），~218 条为 V3→V1 转换（URL action + special_api_action 非空）。passThroughType=1 共 487 条，全部为 V3 透传。总计约 1365 条 API（去重前）。
>
> 同一个 V3 URL 可能对应多条记录（不同角色），分别走透传和转换两种模式，如 `/v3/core/users/login` 同时有 passThroughType=1 和 passThroughType=0 两条记录。
