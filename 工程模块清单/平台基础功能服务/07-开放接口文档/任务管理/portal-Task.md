# portal-Task — 旧门户任务接口（service.portal.Task）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/portal/Task.java`
> 类：`cn.testin.service.portal.Task extends GenericBaseService`（非 Spring MVC Controller，无注解路由）
> 入口方式：旧门户 ApiServlet 网关，`action=portal`（对应包 `cn.testin.service.portal`），`op=Task.方法名` 反射调用；以 `ApiRequest`（内嵌 `reqjson` JSONObject）按方法名派发调用；每个方法返回拼接好的 JSON 字符串（`ApiUtil.getJSONobj / getResult`）
> - **action**: `portal`（对应包 `cn.testin.service.portal`）
> - **入口格式**：`{"op": "Task.方法名", "action": "portal", "data": {...}}`
> 依赖：`IPortalTaskService`（`PortalTaskServiceImpl`）、`IEsPortalSummaryService`（ES 任务汇总）、`IUserInfoDAO`
> 业务：门户侧任务记录的查询（列表/详情/按id集合）、删除（单个/批量）、上报、搜索条件聚合（应用名/版本/渠道/状态分类）、失效重置、ES 刷新。

## 方法列表总表

| #   | 方法              | 说明                             | 主要依赖                        |
| --- | --------------- | ------------------------------ | --------------------------- |
| 1   | list            | 任务列表接口（分页，条件查询，支持 v3 类型/状态转换）  | iesportalsummaryservice（ES） |
| 2   | get             | 获取单个任务详情（可触发 taskSummary 同步校验） | iPortalTaskService + ES     |
| 3   | appNameList     | 某企业某项目组下的应用名列表（搜索用）            | iPortalTaskService          |
| 4   | appChannelList  | 查询任务中应用的渠道信息                   | iPortalTaskService          |
| 5   | report          | 上报任务数据                         | iPortalTaskService          |
| 6   | delete          | 删除单个任务                         | iPortalTaskService          |
| 7   | batchDelete     | 删除多个任务                         | iPortalTaskService          |
| 8   | listBytaskids   | 按任务id集合查询门户任务                  | iPortalTaskService          |
| 9   | suiteSearchList | 应用信息搜索条件集合                     | iPortalTaskService          |
| 10  | vaildTask       | 重置任务状态为无效（按 suiteId）           | iPortalTaskService          |
| 11  | appVersionList  | 某企业某项目组下的应用版本列表（搜索用）           | iPortalTaskService          |
| 12  | statusCategory  | 查询任务状态与结果分类聚合                  | iPortalTaskService          |
| 13  | refresh         | saas 任务同步私有云后刷新到 ES            | iesportalsummaryservice     |

统一返回：JSON 字符串，结构 `{ code, msg, data }`；`data` 内含 `result` / `list` / `object` 等键（`ApiResponse.RES_*` 常量）。
参数校验失败统一返回 `CommonCode.paraInvalid` + 具体提示（如 `eid is invalid`）。
公共分页规则：`page < 1` 归为 1；`pageSize` 为空/非法/超过 `Config.MaxSize` 时取 `Config.MaxSize`。

---

## 1. op=Task.list — 任务列表接口

### 请求格式
{"op": "Task.list", "action": "portal", "data": {"eid": ..., "projectid": ..., "page": ..., "pageSize": ...}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id，`null 或 <= 0` 返回 paraInvalid |
| projectid | Integer | 是 | 项目组id，`null 或 <= 0` 返回 paraInvalid |
| taskid / taskids / noTaskIds | Integer / List / List | 否 | 单任务id / 包含的任务id数组 / 排除的任务id数组 |
| packageName / taskDescr | String | 否 | 包名 / 任务描述（模糊条件） |
| createPerson | String | 否 | 创建人；含 `@` 时按邮箱查 `DbUserInfo` 转成 userId 条件 |
| syspfId / bizCode / taskStatus / execResult | Integer / String / Integer / Integer | 否 | 系统平台 / 测试类型 / 执行状态 / 执行结果 |
| taskType | String | 否 | v3 接口测试类型：`NEW_APP_TEST→JUNIT_TEST`、`NEW_WEB_TEST→WEB_TEST`、`NEW_DESKTOP_TEST→PC_TEST` 转换后作为 bizCode，并标记 `needDealResponse=true` |
| taskExecuteStatus | String | 否 | v3 三端统一状态，配合 bizCode 经 `TaskStatusEnum.getOldStatusByNewStatus(bizCode+"_"+status)` 转回旧状态查询 |
| startTime / endTime | Long | 否 | 时间段（Long） |
| appid / appVersion / channelId / appMd5 | Integer / String / Integer / String | 否 | 应用id / 版本 / 渠道 / 包MD5 |
| suiteId | Integer | 否 | 应用ID（跨平台） |
| includeResult / resultType | String / String | 否 | 结果分类；resultType 经 `TaskResultTypeEnum.getTypeResultMsg` 转 includeResult |
| execStandard | String | 否 | 兼容任务执行策略 |
| quartz | Integer | 否 | 1 = 定时任务（`QUARTZ_JOB`）；定时任务且 taskids 为空时直接返回空成功结果 |
| orderByCol / orderByType | String / String | 否 | 排序字段 / 排序方向 |
| jobId | Integer | 否 | job id |
| page / pageSize | Integer | 否 | 分页，见公共分页规则 |

### 响应结构

`data` 由 `baseListToResData` 从 `BaseList<DbPortalTask>` 转换的分页结构。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | Object | 分页结果对象 |
| data.list | List\<DbPortalTask\> | 门户任务列表 |
| data.page | Integer | 当前页码 |
| data.pageSize | Integer | 每页大小 |
| data.totalRow | Integer | 总行数 |
| data.totalPage | Integer | 总页数 |

### 实现意图

任务列表查询走 **ES**（`iesportalsummaryservice.baseList`，旧代码走 `iPortalTaskService.list` 已注释）；`needDealResponse`（v3 请求）时对返回列表做与 [TaskController](TaskController.md) 第 6 接口一致的转换：旧状态→新状态、bizCode 三端归一（JUNIT_TEST→NEW_APP_TEST 等）。

---

## 2. op=Task.get — 获取单个任务详情

### 请求格式
{"op": "Task.get", "action": "portal", "data": {"eid": ..., "taskid": ...}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| taskid | Integer | 是 | 任务id，空返回 paraInvalid |
| isSyncTaskSummaryInfo | Integer | 否 | >= 1 时触发 taskSummary 同步校验（默认按需） |

### 响应结构

`data.object` = `DbPortalTask`；并从 ES 汇总补 `scriptTotalExecTime`。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | Object | 业务数据对象 |
| data.object | DbPortalTask | 门户任务详情 |
| data.object.scriptTotalExecTime | Long | 脚本总执行时间（来自 ES 汇总） |

### 实现意图

查单任务详情；`isSyncTaskSummaryInfo >= 1` 时执行私有方法 `isSyncTaskSummaryInfo(task)`：解析 `content` 中 deviceTotal/scriptTotal/execStandard/taskSummary，按执行策略计算应有任务总数（fast/data/retention/replace → scriptTotal；normal/script → scriptTotal × deviceTotal），若 `taskSummary.testResult` 的 val 合计与总数不等且状态为完成，则在内存对象上把状态回退为运行中（web 任务 bizCode=4100 用 web_running/web_ok/web_canceled 一套状态）。注意：只改返回对象，**不写库**（相关写库代码均已注释）。

---

## 3. op=Task.appNameList — 应用名列表（搜索条件）

### 请求格式
{"op": "Task.appNameList", "action": "portal", "data": {"eid": ..., "projectid": ...}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid / projectid | Integer / Integer | 是 | 企业id / 项目组id |
| bizCode | String | 否 | 传了则必须 > 0 |
| syspfid | Integer | 否 | 传了则必须 >= 0 |
| suiteId | Integer | 否 | 跨平台应用ID |

### 响应结构

`data.list` = `List<PortalVersionCondition>`（空时返回空数组）。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | Object | 业务数据对象 |
| data.list | List\<PortalVersionCondition\> | 应用名列表 |

### 调用链

`iPortalTaskService.queryAppNameList(projectid, eid, bizCode, syspfid, suiteId)`

---

## 4. op=Task.appChannelList — 任务渠道信息

### 请求格式
{"op": "Task.appChannelList", "action": "portal", "data": {"eid": ..., "projectid": ...}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid / projectid | Integer / Integer | 是 | 企业id / 项目组id |
| bizCode / suiteId / appid / appVersion | String / Integer / Integer / String | 否 | 业务编码 / 应用ID / 应用id / 版本 |

### 响应结构

`data.list` = `List<String>` 渠道列表。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | Object | 业务数据对象 |
| data.list | List\<String\> | 渠道列表 |

### 调用链

`iPortalTaskService.queryAppChannelList(projectid, eid, bizCode, suiteId, appid, appVersion)`

---

## 5. op=Task.report — 上报任务数据

### 请求格式
{"op": "Task.report", "action": "portal", "data": {{...DbPortalTask...}}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| data（整包） | DbPortalTask | 是 | 整包 JSON 由 `DbPortalTask.toBean(reqjson)` 反序列化为门户任务对象 |

### 响应结构

`data.result` = 1 成功 / 0 失败。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | Object | 业务数据对象 |
| data.result | Integer | 1 成功 / 0 失败 |

### 调用链

`iPortalTaskService.report(task)`

---

## 6. op=Task.delete — 删除单个任务

### 请求格式
{"op": "Task.delete", "action": "portal", "data": {"eid": ..., "taskid": ...}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| taskid | Integer | 是 | 任务id |

### 响应结构

`data.result` = 1 / 0。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | Object | 业务数据对象 |
| data.result | Integer | 1 成功 / 0 失败 |

### 调用链

`iPortalTaskService.remove(taskid, eid, true)`

---

## 7. op=Task.batchDelete — 删除多个任务

### 请求格式
{"op": "Task.batchDelete", "action": "portal", "data": {"eid": ..., "projectid": ..., "taskids": [...]}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid / projectid | Integer / Integer | 是 | 企业id / 项目组id |
| taskids | List | 是 | 任务id数组，空数组返回 paraInvalid；空串元素被跳过 |

### 响应结构

`data.result` = 删除条数（Integer）。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | Object | 业务数据对象 |
| data.result | Integer | 删除条数 |

### 调用链

`iPortalTaskService.batchRemove(taskIds, eid, projectid)`

---

## 8. op=Task.listBytaskids — 按任务id集合查询

### 请求格式
{"op": "Task.listBytaskids", "action": "portal", "data": {"eid": ..., "projectid": ..., "tasks": [...]}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid / projectid | Integer / Integer | 是 | 企业id / 项目组id |
| tasks | List | 是 | 任务id数组，空返回 paraInvalid |

### 响应结构

`data` 由 `listToResList` 转换的 `List<DbPortalTask>`。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | List\<DbPortalTask\> | 门户任务列表 |

### 调用链

`iPortalTaskService.listByTaskids(eid, projectid, taskids)`

---

## 9. op=Task.suiteSearchList — 应用信息搜索条件集合

### 请求格式
{"op": "Task.suiteSearchList", "action": "portal", "data": {"eid": ..., "projectid": ..., "bizCode": ...}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid / projectid | Integer / Integer | 是 | 企业id / 项目组id |
| bizCode | String | 是 | 业务编码（null 即 paraInvalid） |

### 响应结构

`data.list` = `List<Integer>`。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | Object | 业务数据对象 |
| data.list | List\<Integer\> | 应用信息搜索条件集合 |

### 调用链

`iPortalTaskService.suiteSearchList(eid, projectid, bizCode)`

---

## 10. op=Task.vaildTask — 重置任务状态为无效

### 请求格式
{"op": "Task.vaildTask", "action": "portal", "data": {"eid": ..., "projectid": ..., "suiteId": ...}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid / projectid | Integer / Integer | 是 | 企业id / 项目组id |
| suiteId | Integer | 是 | 应用ID（跨平台） |

### 响应结构

`data.result` = 影响行数。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | Object | 业务数据对象 |
| data.result | Integer | 影响行数 |

### 调用链

`iPortalTaskService.validTask(eid, projectid, suiteId)`（注意方法名拼写 vaildTask / validTask 不一致，属历史遗留）

---

## 11. op=Task.appVersionList — 应用版本列表（搜索条件）

### 请求格式
{"op": "Task.appVersionList", "action": "portal", "data": {"eid": ..., "projectid": ...}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid / projectid | Integer / Integer | 是 | 企业id / 项目组id |
| bizCode | String | 否 | 传了则必须 > 0 |
| syspfId | Integer | 否 | 传了则必须 >= 0 |
| suiteId / appid | Integer / Integer | 否 | 跨平台应用ID / 应用id |

### 响应结构

`data.list` = `List<String>` 版本列表。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | Object | 业务数据对象 |
| data.list | List\<String\> | 版本列表 |

### 调用链

`iPortalTaskService.queryAppVersionList(projectid, eid, bizCode, syspfid, suiteId, appid)`

---

## 12. op=Task.statusCategory — 任务状态与结果分类聚合

### 请求格式
{"op": "Task.statusCategory", "action": "portal", "data": {"eid": ..., "projectid": ..., "bizCode": ...}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid / projectid | Integer / Integer | 是 | 企业id / 项目组id |
| bizCode | String | 是 | 业务编码（必须 > 0） |
| suiteId / appid / appVersion / syspfId | Integer / Integer / String / Integer | 否 | 组装为 conditionMap |

### 响应结构

`data.object` = `Map<String, JSONArray>`（空时返回空对象）。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | Object | 业务数据对象 |
| data.object | Map\<String, JSONArray\> | 任务状态与结果分类聚合 |

### 调用链

`iPortalTaskService.queryStatusAndCategory(conditionMap)`

---

## 13. op=Task.refresh — 任务刷新到 ES

### 请求格式
{"op": "Task.refresh", "action": "portal", "data": {"eid": ..., "taskid": ...}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| taskid | Integer | 是 | 任务id |

### 响应结构

`data.result` = 1 / 0。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 消息 |
| data | Object | 业务数据对象 |
| data.result | Integer | 1 成功 / 0 失败 |

### 实现意图

saas 任务同步到私有云后，触发该任务在 ES 汇总（EsPortalSummary）中的刷新。

### 调用链

`iesportalsummaryservice.refresh(eid, taskid)`

---

## 备注

- 本类与 [TaskController](TaskController.md)（`/task/**` Spring MVC 接口）是两套并存的入口：本类为旧门户 JSON 派发风格，参数全部从 `reqjson` 手工取并逐个校验；新接口走 DTO + `@Valid`。
- 三端统一转换（bizCode 归一、新旧状态映射）在本类 `list` 与 `TaskController.getTasksByJobId` 中各有一份重复实现，改动时需同步两处。
- `get` 中通过 `SpringUtil.getBean(iesportalsummaryservice.getClass())` 再取一次 Bean，等价于直接用字段，属冗余写法。
- `isSyncTaskSummaryInfo` 中的空指针保护存在缺陷：`task == null` 判断在打日志时已先解引用 `task.getTaskid()`；且方法名为 is 开头但实际是校验+回退状态。
- 依赖 DAO/Service 字段统一继承自 `GenericBaseService`（`iPortalTaskService`、`iesportalsummaryservice`、`iuserinfodao` 等均以 `SpringUtil.getBean` 注入，非 Spring 容器管理本类实例）。

相关文档：[00-分支索引](00-分支索引.md) · [ExecuteRecordController](../测试计划/ExecuteRecordController.md)
