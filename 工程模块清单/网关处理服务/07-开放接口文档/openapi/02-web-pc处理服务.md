---
module: real-web
type: 开放接口文档
source: db_mcfg + 代码
---

# web-pc处理服务 OpenAPI 接口文档

> 代码仓库：`real-web`（分支 `syy.release.z7.8.1.0`）。
> V1 action/op 路由类在 `cn.testin.realweb.service.<action>.<类名>`（如 `cn.testin.realweb.service.task.Task`）；V3 接口在 `cn.testin.realweb.mvc.controller` 包（`@RestController`）。
> 网关路由：模块 `realweb` / `realpc` / `common` → `http://testin-aio-real-web:8080`。

## 通用返回结构

**V1（ApiServlet，action/op 模式）**

```json
{ "code": 0, "message": "成功", "data": { ... } }
```

`code=0` 表示成功（`CommonCode.success`）。业务数据放在 `data` 内，常见字段：

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer/String | 操作结果（任务id、影响行数、1/0 等，随接口而定） |
| data.objInfo | JSONObject | 单对象结果 |
| data.list | JSONArray | 列表结果 |
| data.page / pageSize / totalPage / totalRow | Integer/Long | 分页信息（分页接口） |

**V3（Spring MVC，透传模式）**

统一封装 `ResponseResult<T>`（`cn.testin.realweb.utils.ResponseResult`），JSON 结构：

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功（`CommonConstant.SUCCESS_CODE`） |
| msg | String | 提示信息（成功为「成功」） |
| data | T | 业务数据，类型随接口而定（`BaseResponseDTO` / `PageResponseDTO<T>` / `ResultListResponseDTO<T>` 等） |

- `BaseResponseDTO`：`{ "result": Integer }`（更新/删除类接口返回处理数据量）。
- `BaseResultStrResponseDTO`：`{ "result": String }`。
- `PageResponseDTO<T>`：`{ page, pageSize, totalPage, totalRow, list }`。
- `ResultListResponseDTO<T>`：`{ list }`。

## 通用约定

- 转发模式标记：🟢 V1 原生 / 🔵 V3 透传 / 🟡 V3→V1 转换。
- `needLogin=1` 的接口网关侧需额外携带 `sid`；所有 HTTP 模块均需 `apikey` + `mkey`。
- V1 请求为 `POST`，Body 内 `action` + `op` + 业务字段；V3 请求 GET 参数走 query，POST 走 `@RequestBody` JSON。
- 「必填」依据：V1 看 `reqJson.isNull/optInt/optString` 判空与 `Assert.hasText`；V3 看 `@RequestBody` DTO 的 `@NotNull/@NotBlank/@NotEmpty` 及 Controller 内显式 `StringUtils.isBlank` 校验。
- 路径变量以代码实际变量名标注（如 `{task_id}`），网关路由表中的 `{param1}` 为占位符。
- 字段无法从代码确认的一律标注「（代码未确认）」。

---

## 一、任务管理（V1 原生，action=task，mkey=realweb/realpc）

> 路由类 `cn.testin.realweb.service.task.Task`。web 任务 taskid 以 `wt` 开头、pc 任务以 `pt` 开头。

### 新增测试任务 `Task.create`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.create` |
| 鉴权 | needLogin=1 |
| 说明 | 新增 web/pc 测试任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目组id |
| userid | Integer | 是 | 用户id |
| bizCode | Integer | 是 | 业务编码（非空） |
| taskDescr | String | 是 | 任务描述 |
| browsers | JSONArray | 是(与pcs二选一) | 浏览器列表（与 pcs 至少一个非空） |
| pcs | JSONArray | 是(与browsers二选一) | PC 列表（与 browsers 至少一个非空） |
| scripts | JSONArray | 是 | 脚本信息列表 |
| apkey | String | 否 | 由网关 apikey 自动填充 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | String | 新建任务id（taskid） |

> 代码出处：`cn.testin.realweb.service.task.Task.create`

### 终止测试 `Task.cancel`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.cancel` |
| 鉴权 | needLogin=1 |
| 说明 | 终止测试任务（支持按子任务、任务组） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目组id |
| taskid | String | 是 | 任务id |
| subtaskid | String | 否 | 子任务id |
| taskGroup | JSONObject | 否 | 恒生任务组（内部 `id` 必填） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 终止影响数 |

> 代码出处：`cn.testin.realweb.service.task.Task.cancel`

### 批量终止测试 `Task.batchCancel`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.batchCancel` |
| 鉴权 | needLogin=1 |
| 说明 | 批量取消/终止测试任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目组id |
| taskids | JSONArray | 是 | 任务id列表 |
| taskGroup | JSONObject | 否 | 恒生任务组（内部 `id` 必填） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 成功取消的任务数 |

> 代码出处：`cn.testin.realweb.service.task.Task.batchCancel`

### 任务详情 `Task.detail`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.detail` |
| 鉴权 | needLogin=1 |
| 说明 | 查询任务详情（含脚本、浏览器/上位机、报告关联信息） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| skey | String | 否 | 分享 key，与 taskid 二选一 |
| keywords | JSONArray | 否 | 指定返回的字段名列表 |
| scriptStatuses | JSONArray | 否 | 脚本执行状态过滤 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.objInfo | JSONObject | PmTaskDetail 任务详情全量字段（taskid/projectid/eid/userid/bizCode/testType/execStatus/scripts/browsers/pcs 等） |
| data.objInfo.relations | JSONObject | 关联测试计划执行记录（recordRelation 结果） |

> 代码出处：`cn.testin.realweb.service.task.Task.detail`

### 更新任务 `Task.modify`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.modify` |
| 鉴权 | needLogin=1 |
| 说明 | 更新任务详情（当前仅报告总结 summarize） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| summarize | String | 否 | 报告总结 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`cn.testin.realweb.service.task.Task.modify`

### 补测 `Task.repeatTest`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.repeatTest` |
| 鉴权 | needLogin=1 |
| 说明 | 浏览器/脚本批量补测 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 补测任务id |
| subtasks | JSONArray | 是 | 补测子任务id列表 |
| browsers | JSONArray | 否 | 补测浏览器（web 任务；含 deviceSign/ip/osName/source/type/version） |
| pcs | JSONArray | 否 | 补测上位机（pc 任务，taskid 以 `pt` 开头时读取） |
| subsubtaskinfo | JSONObject | 否 | 脚本补测信息（subtaskid → [subsubtaskid/scriptid/scriptNo/orderNum]） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`cn.testin.realweb.service.task.Task.repeatTest`

### 脚本执行详情 `Task.scriptRunInfos`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.scriptRunInfos` |
| 鉴权 | needLogin=1 |
| 说明 | 分页查询脚本执行详情 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| skey | String | 否 | 分享 key |
| resultCategorys | String | 否 | 结果分类（JSON 数组字符串，>100 归入 errorCauseTypeIds） |
| errorCauseTypeIds | String | 否 | 自定义错误类型id（JSON 数组字符串） |
| subtaskids | JSONArray | 否 | 子任务id集合 |
| scriptDescr | String | 否 | 脚本描述 |
| scriptNo | Integer | 否 | 脚本编号 |
| errorMsg | String | 否 | 错误信息 |
| type | String | 否 | 类型 |
| version | String | 否 | 版本 |
| systemBitness / cpuArch / systemVersion / systemName / systemType | String | 否 | 操作系统过滤条件 |
| page | Integer | 否 | 页码（默认 1） |
| pageSize | Integer | 否 | 每页条数（默认/上限 100） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.objInfo.scriptRunInfos | JSONObject | BaseList\<PmScriptRunInfo\>（list/total 等分页结构） |

> 代码出处：`cn.testin.realweb.service.task.Task.scriptRunInfos`

### 浏览器执行详情 `Task.browserRunInfos`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.browserRunInfos` |
| 鉴权 | needLogin=1 |
| 说明 | 分页查询浏览器执行详情（web 任务） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| browserTypes / bowserTypes | JSONArray | 否 | 浏览器类型列表 |
| ips | JSONArray | 否 | 浏览器 ip 列表 |
| osNames | JSONArray | 否 | 操作系统名称列表 |
| resultCategorys | JSONArray | 否 | 结果分类（>100 归入 errorCauseTypes） |
| versions | JSONArray | 否 | 浏览器版本列表 |
| page / pageSize | Integer | 否 | 分页（默认 1 / 100） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.objInfo | JSONObject | 浏览器执行详情结果 Map |

> 代码出处：`cn.testin.realweb.service.task.Task.browserRunInfos`

### PC 执行详情 `Task.clientRunInfos`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.clientRunInfos` |
| 鉴权 | needLogin=1 |
| 说明 | 分页查询 pc 上位机执行详情（mkey=realpc） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| systemVersions | JSONArray | 否 | 系统版本列表 |
| systemNames | JSONArray | 否 | 系统名称列表 |
| systemTypes | JSONArray | 否 | 系统类型列表 |
| ips | JSONArray | 否 | 上位机 ip 列表 |
| resultCategorys | JSONArray | 否 | 结果分类（>100 归入 errorCauseTypes） |
| page / pageSize | Integer | 否 | 分页（默认 1 / 100） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.objInfo | JSONObject | pc 执行详情结果 Map |

> 代码出处：`cn.testin.realweb.service.task.Task.clientRunInfos`

### 重新发送邮件 `Task.sendEmail`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.sendEmail` |
| 鉴权 | needLogin=1 |
| 说明 | 重新发送任务结果邮件 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 发送结果 |

> 代码出处：`cn.testin.realweb.service.task.Task.sendEmail`

### 修改错误定位 `Task.modifyErrorMsg`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.modifyErrorMsg` |
| 鉴权 | needLogin=1 |
| 说明 | 更新脚本执行记录的错误定位信息 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| subtaskid | String | 是 | 子任务id |
| subsubtaskid | String | 是 | 子子任务id |
| errorMsg | String | 否 | 错误信息 |
| userId | Integer | 否 | 用户id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 更新结果 |

> 代码出处：`cn.testin.realweb.service.task.Task.modifyErrorMsg`

### 执行概况查询条件 `Task.runInfoConditions`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.runInfoConditions` |
| 鉴权 | needLogin=1 |
| 说明 | 获取执行概况查询条件（路由表标注对应 TestProcess，实际实现在 Task） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| type | String | 是 | `script` 查脚本条件，其它查设备条件 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.objInfo | JSONObject | 查询条件 JSON |

> 代码出处：`cn.testin.realweb.service.task.Task.runInfoConditions`

### 触发任务执行 `Task.execute`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.execute` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 触发任务执行 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 执行结果 |

> 代码出处：`cn.testin.realweb.service.task.Task.execute`

### 暂停任务下发 `Task.pause`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.pause` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 暂停任务下发 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 执行结果 |

> 代码出处：`cn.testin.realweb.service.task.Task.pause`

### 恢复任务下发 `Task.resume`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.resume` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 恢复任务下发 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 执行结果 |

> 代码出处：`cn.testin.realweb.service.task.Task.resume`

### 验证 taskid/skey 有效性 `Task.verification`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=Task.verification` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 验证 taskid 和 skey 的有效性 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| skey | String | 否 | 分享 key |
| userid / eid / userprojectids | — | 否 | 权限校验用（taskInfoSupplement 内） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 1 有效 |

> 代码出处：`cn.testin.realweb.service.task.Task.verification`

---

## 二、测试过程与结果上报（V1 原生，action=task）

> 执行机（上位机/浏览器）上报接口。

### 测试过程上报 `TestProcess.report`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=TestProcess.report` |
| 鉴权 | （代码未确认，路由表未列出；执行机上报） |
| 说明 | 执行机测试过程上报 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskAction | String | 是 | 过程类型（`REPORT` 等，见 TaskAction 枚举） |
| content | JSONObject | 是(条件) | 过程内容（仅当 taskAction=REPORT 时必填） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 上报结果 |

> 代码出处：`cn.testin.realweb.service.task.TestProcess.report`

### 测试结果上报 `TestResult.report`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=task&op=TestResult.report` |
| 鉴权 | （代码未确认，路由表未列出；执行机上报） |
| 说明 | 执行机测试结果上报与解析 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskResult | JSONObject | 是 | 结果对象（内部 `taskid` 必填） |
| resultFiles | JSONObject | 否 | 各结果文件 JSON |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`cn.testin.realweb.service.task.TestResult.report`

---

## 三、报告查询（V1 原生，action=report）

> 路由类 `cn.testin.realweb.service.report.Report`。
> 注：路由表标注的 `Report.netReqDetail` 在 `Report.java` 中无对应方法，实际网络性能接口为 `stepInternetInfo` 与 `performanceCondition`（`netReqDetail` 已不存在于当前代码）。

### 报告详情列表 `Report.list`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=report&op=Report.list` |
| 鉴权 | needLogin=1 |
| 说明 | 分页查询报告详情（脚本级结果列表） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| subtaskid | String | 否 | 子任务id |
| needTestCases | Integer | 否 | 是否返回 testCases（默认 0 排除） |
| resultCategorys | String | 否 | 结果分类（JSON 数组字符串，>100 归入 errorCauseTypeIds） |
| errorCauseTypeIds | String | 否 | 自定义错误类型id |
| keywords | JSONArray | 否 | 关键字查询 |
| subsubTaskIds | JSONArray | 否 | 子子任务id列表 |
| page | Integer | 否 | 页码（默认 1） |
| pageSize | Integer | 否 | 每页条数（默认/上限 200） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.list | JSONArray | PmReportDetail 列表 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |

> 代码出处：`cn.testin.realweb.service.report.Report.list`

### 脚本步骤菜单树 `Report.scriptSteps`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=report&op=Report.scriptSteps` |
| 鉴权 | needLogin=1 |
| 说明 | 左侧菜单树——脚本步骤接口 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| subtaskid | String | 是 | 子任务id |
| subsubtaskid | String | 否 | 子子任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.objInfo | JSONObject | 脚本树详情 Map |

> 代码出处：`cn.testin.realweb.service.report.Report.scriptSteps`

### 脚本步骤详情 `Report.stepdetail`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=report&op=Report.stepdetail` |
| 鉴权 | needLogin=1 |
| 说明 | 查询脚本步骤详情 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| subtaskid | String | 是 | 子任务id |
| subsubtaskid | String | 否 | 子子任务id |
| scriptTag | String | 是 | 脚本标签 |
| stepid | Integer | 是 | 步骤id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.objInfo | JSONObject | 步骤详情 Map |

> 代码出处：`cn.testin.realweb.service.report.Report.stepdetail`

### 脚本执行概况 `Report.taskSummary`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=report&op=Report.taskSummary` |
| 鉴权 | needLogin=1 |
| 说明 | 脚本执行概况（任务汇总） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.objInfo | JSONObject | PmTaskSummary 汇总数据（无数据返回 `{}`） |

> 代码出处：`cn.testin.realweb.service.report.Report.taskSummary`

### 测试过程 `Report.testProcess`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=report&op=Report.testProcess` |
| 鉴权 | needLogin=1 |
| 说明 | 查询测试过程记录 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| subsubtaskid | String | 是 | 子子任务id |
| name | String | 是 | 名称 |
| stage | String | 是 | 阶段 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.list | JSONArray | List\<TestProcess\> |

> 代码出处：`cn.testin.realweb.service.report.Report.testProcess`

### 脚本业务检查点 `Report.scriptCheckInfos`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=report&op=Report.scriptCheckInfos` |
| 鉴权 | needLogin=1 |
| 说明 | 获取脚本中的业务检查点 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| subtaskid | String | 是 | 子任务id |
| subsubtaskid | String | 是 | 子子任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.objInfo | JSONObject | Map\<String, List\<ScriptCheckInfo\>\> |

> 代码出处：`cn.testin.realweb.service.report.Report.scriptCheckInfos`

### 脚本执行概况（单机页） `Report.scriptRunInfoSummary`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=report&op=Report.scriptRunInfoSummary` |
| 鉴权 | needLogin=1 |
| 说明 | 单机页脚本执行概况 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| subtaskid | String | 是 | 子任务id |
| subsubtaskid | String | 否 | 子子任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.objInfo | JSONObject | 脚本执行概况 Map |

> 代码出处：`cn.testin.realweb.service.report.Report.scriptRunInfoSummary`

### 报告设备信息（浏览器） `Report.getBrowserInfo`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=report&op=Report.getBrowserInfo` |
| 鉴权 | needLogin=1 |
| 说明 | 报告详情获取浏览器设备信息 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| subtaskid | String | 是 | 子任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.objInfo | JSONObject | BrowserInfo 浏览器信息 |

> 代码出处：`cn.testin.realweb.service.report.Report.getBrowserInfo`

### 报告设备信息（上位机） `Report.getClientInfo`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=report&op=Report.getClientInfo` |
| 鉴权 | needLogin=1 |
| 说明 | 报告详情获取 pc 上位机设备信息 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| subtaskid | String | 是 | 子任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.objInfo | JSONObject | ClientInfo 上位机信息 |

> 代码出处：`cn.testin.realweb.service.report.Report.getClientInfo`

### 单步骤网络信息解析 `Report.stepInternetInfo`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=report&op=Report.stepInternetInfo` |
| 鉴权 | needLogin=1 |
| 说明 | 单步骤的网络信息解析（网络性能明细） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| subtaskid | String | 是 | 子任务id |
| subsubtaskid | String | 是 | 子子任务id |
| scriptTag | String | 是 | 脚本标签 |
| stepid | Integer | 是 | 步骤id |
| initiatorTypes | JSONArray | 否 | 资源类型过滤 |
| statusCodes | JSONArray | 否 | 状态码过滤 |
| requestUrl | String | 否 | 请求地址过滤 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.list | JSONArray | List\<NetPerformance\> |

> 代码出处：`cn.testin.realweb.service.report.Report.stepInternetInfo`

### 网络性能查询条件 `Report.performanceCondition`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=report&op=Report.performanceCondition` |
| 鉴权 | needLogin=1 |
| 说明 | 获取网络性能数据的查询条件 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| subtaskid | String | 是 | 子任务id |
| subsubtaskid | String | 是 | 子子任务id |
| scriptTag | String | 是 | 脚本标签 |
| stepid | Integer | 是 | 步骤id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.objInfo | JSONObject | 查询条件 Map |

> 代码出处：`cn.testin.realweb.service.report.Report.performanceCondition`

### 修改结果分类 `Report.modifyResultCategory`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=report&op=Report.modifyResultCategory` |
| 鉴权 | needLogin=1 |
| 说明 | 更改子子任务的结果分类 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| subtaskid | String | 是 | 子任务id |
| subsubtaskid | String | 否 | 子子任务id（空为设备级，非空为脚本级） |
| resultCategory | Integer | 是 | 结果分类编码（ResultCategoryEnum） |
| userid | Integer | 否 | 用户id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 1 成功 / 0 失败 |

> 代码出处：`cn.testin.realweb.service.report.Report.modifyResultCategory`

### 获取报告 url `Report.url`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=report&op=Report.url` |
| 鉴权 | needLogin=1 |
| 说明 | 根据任务id获取报告 url（wt→web、pt→pc） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| projectid | Integer | 是 | 项目组id |
| subtaskid | String | 否 | 子任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | String | 报告 url |

> 代码出处：`cn.testin.realweb.service.report.Report.url`

### Excel 导出 `Excel.reportExcel`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=report&op=Excel.reportExcel` |
| 鉴权 | needLogin=1 |
| 说明 | 报告 Excel 异步生成与获取（轮询） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是(与skey二选一) | 任务id（或提供 skey） |
| skey | String | 否 | 分享 key |
| eid | Integer | 否 | 企业id |
| userid | Integer | 否 | 用户id |
| userprojectids | JSONArray | 否 | 用户项目id列表（权限校验） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 1 成功 / 0 生成中（继续轮询） / -1 生成失败 |
| data.excelUrl | String | Excel 下载地址（result=1 时返回） |

> 代码出处：`cn.testin.realweb.service.report.Excel.reportExcel`

### PDF 导出 `Pdf.parse`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=report&op=Pdf.parse` |
| 鉴权 | needLogin=1 |
| 说明 | HTML 报告转 PDF（队列异步，按 md5 去重） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| url | String | 是 | 待转换的报告 url |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.objInfo.code | Integer | -1 失败 / 0 等待 / 1 成功 |
| data.objInfo.md5Key | String | url 的 md5 标识 |
| data.objInfo.targetUrl | String | 目标 url |

> 代码出处：`cn.testin.realweb.service.report.Pdf.parse`

---

## 四、数据源（V1 原生，action=dataSource，needLogin=0）

> 路由类 `cn.testin.realweb.service.dataSource.ParamDataSource`。

### 获取参数表数据 `ParamDataSource.getParamTableData`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=dataSource&op=ParamDataSource.getParamTableData` |
| 鉴权 | needLogin=0 |
| 说明 | 参数化数据源表格数据查询 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sourceId | String | 是 | 数据源id |
| projectId | Integer | 是 | 项目id |
| scriptNo | Integer | 是 | 脚本编号 |
| tagList | JSONArray | 否 | 标签id列表 |
| skipTagList | JSONArray | 否 | 跳过标签id列表 |
| oldTaskId | String | 否 | 原任务id |
| scriptStatus | JSONArray | 否 | 脚本状态列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | JSONArray | 参数表数据列表 |

> 代码出处：`cn.testin.realweb.service.dataSource.ParamDataSource.getParamTableData`

### 分配默认数据源参数 `ParamDataSource.getDefaultDataSourceParam`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=dataSource&op=ParamDataSource.getDefaultDataSourceParam` |
| 鉴权 | needLogin=0 |
| 说明 | 根据脚本与设备分配默认数据源参数 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sourceId | Integer | 是 | 数据源id（作 parentId） |
| projectId | Integer | 是 | 项目id |
| eid | Integer | 是 | 企业id |
| paramStrategy | Integer | 是 | 参数策略 |
| scriptNos | JSONArray | 是 | 脚本编号列表 |
| devices | JSONArray | 是 | 设备列表 |
| tagList | JSONArray | 否 | 标签id列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | JSONArray | 分配的默认数据源参数列表 |

> 代码出处：`cn.testin.realweb.service.dataSource.ParamDataSource.getDefaultDataSourceParam`

---

## 五、定时任务（V1 原生，action=quartz）

> 路由类 `cn.testin.realweb.service.quartz.Quartz` 与 `cn.testin.realweb.service.quartz.Report`。公共必填参数：`eid`、`projectid`、`userid`、`userName`、`businessType`、`bizCode`（见 `verifyParams`）；按业务类型追加 `scripts`/`browsers`/`pcs`。
> 鉴权：路由表未列出，默认按 needLogin=1 处理。

### 新增定时任务 `Quartz.add`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.add` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 新增定时任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| projectid / projectId | Integer | 是 | 项目id |
| userid / userId | Integer | 是 | 用户id |
| userName | String | 是 | 用户名 |
| businessType | Integer | 是 | 业务类型（app/web/mcpc/cross） |
| bizCode | Integer | 是 | 业务编码 |
| taskDescr / desc | String | 是 | 任务描述 |
| scripts | JSONArray | 是(按类型) | 脚本列表（app/web/mcpc/cross 均需） |
| browsers | JSONArray | 是(web) | 浏览器列表（businessType=web 时） |
| pcs | JSONArray | 是(mcpc) | PC 列表（businessType=mcpc 时） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 新增结果（jobId） |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.add`

### 修改定时任务 `Quartz.update`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.update` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 修改定时任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobId | Integer | 是 | 定时任务id |
| 其余参数 | — | 是 | 同 `Quartz.add`（eid/projectid/userid/userName/businessType/bizCode/taskDescr/scripts/browsers/pcs） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 更新结果 |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.update`

### 定时任务列表 `Quartz.list`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.list` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 分页查询定时任务列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目id |
| page | Integer | 是 | 页码 |
| pageSize | Integer | 是 | 每页条数 |
| businessType | String | 是 | 业务类型 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | JSONObject | PageUtils 分页结果 |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.list`

### 删除定时任务 `Quartz.delete`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.delete` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 删除定时任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobId | Integer | 是 | 定时任务id |
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目id |
| businessType | String | 是 | 业务类型 |
| userid | Integer | 否 | 用户id |
| username | String | 否 | 用户名 |
| deleteRecords | Integer | 否 | 是否同时删除执行记录（默认 0） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 删除结果 |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.delete`

### 批量删除定时任务 `Quartz.batchDelete`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.batchDelete` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 批量删除定时任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobIds | JSONArray | 是 | 定时任务id列表 |
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目id |
| businessType | String | 是 | 业务类型 |
| userid / username / deleteRecords | — | 否 | 同 `Quartz.delete` |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 删除结果 |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.batchDelete`

### 暂停定时任务 `Quartz.stop`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.stop` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 暂停定时任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobId | Integer | 是 | 定时任务id |
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目id |
| userid | Integer | 否 | 用户id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 结果 |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.stop`

### 批量暂停定时任务 `Quartz.batchStop`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.batchStop` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 批量暂停定时任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobIds | JSONArray | 是 | 定时任务id列表 |
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目id |
| userid | Integer | 否 | 用户id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 成功暂停数量 |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.batchStop`

### 恢复定时任务 `Quartz.reset`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.reset` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 恢复定时任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobId | Integer | 是 | 定时任务id |
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目id |
| userid | Integer | 否 | 用户id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 结果 |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.reset`

### 批量恢复定时任务 `Quartz.batchReset`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.batchReset` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 批量恢复定时任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobIds | JSONArray | 是 | 定时任务id列表 |
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目id |
| userid | Integer | 否 | 用户id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 成功恢复数量 |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.batchReset`

### 立即执行定时任务 `Quartz.start`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.start` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 手动立即执行定时任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobId | String | 是 | 定时任务id |
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目id |
| businessType | Integer | 是 | 业务类型 |
| extendedChannel | String | 否 | 扩展渠道 |
| extendedChannelUrl | String | 否 | 扩展渠道 url |
| userid | Integer | 否 | 用户id |
| username | String | 否 | 用户名 |
| checkDeviceStatus | Integer | 否 | 是否检查设备状态（默认 0） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | String | 执行结果（任务id） |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.start`

### 批量立即执行 `Quartz.batchStart`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.batchStart` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 批量立即执行定时任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobIds | JSONArray | 是 | 定时任务id列表 |
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目id |
| businessType | Integer | 是 | 业务类型 |
| extendedChannel / extendedChannelUrl / userid / username / checkDeviceStatus | — | 否 | 同 `Quartz.start` |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | JSONArray | 执行结果（任务id）列表 |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.batchStart`

### 定时任务详情参数 `Quartz.quartzJobParams`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.quartzJobParams` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 查询定时任务详细信息（修改时回显） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobId | Integer | 是 | 定时任务id |
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目id |
| businessType | String | 是 | 业务类型 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | JSONObject | 定时任务详情 Map |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.quartzJobParams`

### 定时任务执行记录列表 `Quartz.listRealTask`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.listRealTask` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 查询定时任务执行记录列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| jobId | String | 是 | 定时任务id |
| projectid | Integer | 是 | 项目id |
| page | Integer | 是 | 页码 |
| pageSize | Integer | 是 | 每页条数 |
| businessType | String | 是 | 业务类型 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | JSONObject | PageUtils 分页结果 |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.listRealTask`

### 条件查询定时任务 `Quartz.conditionalQuery`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.conditionalQuery` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 条件查询定时任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| page | Integer | 是 | 页码 |
| pageSize | Integer | 是 | 每页条数 |
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目id |
| businessType | String | 是 | 业务类型 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | JSONObject | PageUtils 分页结果 |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.conditionalQuery`

### 定时任务重测 `Quartz.retest`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.retest` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 定时任务重测 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目id |
| taskId | String | 是 | 任务id |
| businessType | Integer | 是 | 业务类型 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | JSONObject | 重测结果 |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.retest`

### 合并脚本组 `Quartz.createScriptGroup`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.createScriptGroup` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 将提测的脚本及脚本组合并成新的脚本组 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| businessType | Integer | 是 | 业务类型 |
| scripts | JSONArray | 是 | 脚本列表 |
| groupDesc | String | 是 | 脚本组描述 |
| scriptType | Integer | 是 | 脚本类型 |
| userId | Integer | 是 | 用户id |
| projectId | Integer | 是 | 项目id |
| userName | String | 是 | 用户名 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 合并结果（新脚本组id） |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.createScriptGroup`

### 获取脚本组脚本id列表 `Quartz.getScriptGroupScript`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Quartz.getScriptGroupScript` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 查询所有定时任务中引用到的脚本id列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| （无必填参数） | — | 否 | 无 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.list | JSONArray | 脚本id列表（String） |

> 代码出处：`cn.testin.realweb.service.quartz.Quartz.getScriptGroupScript`

### 获取任务报告 url（定时任务场景） `Report.url`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=quartz&op=Report.url` |
| 鉴权 | needLogin=1（路由表未列出） |
| 说明 | 定时任务场景获取任务报告 url |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskid | String | 是 | 任务id |
| projectid | Integer | 是 | 项目组id |
| businessType | String | 否 | 业务类型（默认 2=web；app 走 app 报告） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | String | 报告 url |

> 代码出处：`cn.testin.realweb.service.quartz.Report.url`

---

## 六、common 域（V1 原生，mkey=common）

### 获取当前时间 `System.currentTimeMillis`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟢 |
| HTTP 方法 | POST（action/op） |
| 路径 | `action=common&op=System.currentTimeMillis` |
| 鉴权 | needLogin=1 |
| 说明 | 获取服务器当前时间戳 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| （无业务参数） | — | 否 | 无 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| （代码未确认） | — | 返回当前时间戳，real-web 仓库未找到 `common.System` 类，可能由 testin-core 处理 |

> 代码出处：（代码未确认，real-web 仓库无对应类）

---

## 七、V3 透传 — 基础设施（HeartBeatController）

> `cn.testin.realweb.mvc.controller.HeartBeatController`，`@RequestMapping("/realweb")`。

### 健康检查 `GET /v3/realweb/heartbeat/check`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realweb/heartbeat/check` |
| 鉴权 | needLogin=0 |
| 说明 | 检测 mongo/redis/mysql 连接是否正常 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| （无参数） | — | 否 | 无 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功（异常时返回 500 + ResponseResult.error） |
| data | JSONObject | 空对象 `{}` |

> 代码出处：`cn.testin.realweb.mvc.controller.HeartBeatController.check`

---

## 八、V3 透传 — 问题分析报告（ProblemAnalysisReportController）

> `cn.testin.realweb.mvc.controller.ProblemAnalysisReportController`，`@RequestMapping("/realweb")`。

### 问题分析脚本列表 `POST /v3/realweb/report/script_list`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realweb/report/script_list` |
| 鉴权 | needLogin=1 |
| 说明 | 获取脚本任务的问题分析报告列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskId | String | 是 | 任务id（Controller 内判空） |
| projectId | Integer | 否 | 项目id |
| page | Integer | 否 | 页码（默认 1） |
| pageSize | Integer | 否 | 每页数量（默认 50） |
| scriptNo | String | 否 | 脚本编号 |
| scriptName | String | 否 | 脚本名称 |
| testResult | String | 否 | 测试结果过滤 |
| errorCauseTypeId | Integer | 否 | 自定义错误类型id |
| customizeErrorMsg | String | 否 | 自定义错误信息 |
| deviceId | String | 否 | 设备id |
| systemError | String | 否 | 系统错误 |
| isAnalyze | Boolean | 否 | 是否已分析 |
| resultCategory | String | 否 | 结果分类 |
| deviceType | String | 否 | 设备类型 |
| deviceVersion | String | 否 | 设备版本 |
| deviceIp | String | 否 | 设备ip |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |
| data.list.taskId | String | 任务id |
| data.list.subTaskId | String | 子任务id |
| data.list.subSubTaskId | String | 子子任务id |
| data.list.scriptNo | Integer | 脚本编号 |
| data.list.scriptName | String | 脚本名称 |
| data.list.testResult | Integer | 执行结果 |
| data.list.inputParams | String | 执行概要数据 |
| data.list.reportDevice | JSONObject | 设备信息 |
| data.list.resultCategory | Integer | 结果分类 |
| data.list.errorCauseTypeId | Integer | 错误原因类别 |
| data.list.errorCauseMessage | String | 系统级错误信息 |
| data.list.customizeErrorCauseMessage | String | 用户自定义错误信息 |

> 代码出处：`cn.testin.realweb.mvc.controller.ProblemAnalysisReportController.getNeedAnalysisScriptList`

### 报告步骤信息 `GET /v3/realweb/report/steps`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realweb/report/steps` |
| 鉴权 | needLogin=1 |
| 说明 | 报告步骤信息（路由表标注，对应 ProblemAnalysisReportController） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| （代码未确认） | — | 否 | real-web 代码中未找到 `/report/steps` 映射，可能已废弃或由其它服务处理 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| （代码未确认） | — | 无对应实现 |

> 代码出处：（代码未确认，无对应 Controller 方法）

### 错误步骤详情列表 `GET /v3/realweb/report/error_step_report_detail_list`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realweb/report/error_step_report_detail_list` |
| 鉴权 | needLogin=1 |
| 说明 | 获取错误步骤报告详情列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 任务id |
| subsubtask_id | String | 是 | 子子任务id |
| size | Integer | 否 | 返回条数（默认 5） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.list | JSONArray | 错误步骤详情列表（List\<Map\<String,Object\>\>） |
| data.totalRow | Integer | 总行数（= list.size()） |

> 代码出处：`cn.testin.realweb.mvc.controller.ProblemAnalysisReportController.getErrorScriptReportDetailList`

### 任务设备列表 `GET /v3/realweb/report/device_list/{task_id}`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realweb/report/device_list/{task_id}` |
| 鉴权 | needLogin=1 |
| 说明 | 根据任务id获取当前任务用到的设备列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 任务id（路径变量） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data | JSONArray | 设备列表 List\<Map\<String,String\>\> |

> 代码出处：`cn.testin.realweb.mvc.controller.ProblemAnalysisReportController.getDeviceList`

### 刷新执行概要 `POST /v3/realweb/report/refresh_report_input_param`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realweb/report/refresh_report_input_param` |
| 鉴权 | needLogin=1 |
| 说明 | 更新脚本执行列表的执行概要数据 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskId | String | 否 | 任务id |
| subTaskId | String | 否 | 子任务id |
| subSubTaskId | String | 否 | 子子任务id |
| taskIds | JSONArray | 否 | 任务id列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 更新结果 |

> 代码出处：`cn.testin.realweb.mvc.controller.ProblemAnalysisReportController.updateReportInputParam`

---

## 九、V3 透传 — 报告查询（ReportController）

> `cn.testin.realweb.mvc.controller.ReportController`，`@RequestMapping("report")`。以下接口路由表未单独列出，按代码路径记录。

### 补测历史记录 `GET /report/report/{task_id}`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/report/report/{task_id}` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 查询补测历史记录 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 任务id（路径变量） |
| sub_sub_task_id | String | 否 | 子子任务id |
| start_time | Long | 否 | 开始时间 |
| end_time | Long | 否 | 结束时间 |
| page | Integer | 否 | 页码（默认 1） |
| page_size | Integer | 否 | 每页大小（默认 20） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.page / pageSize / totalPage / totalRow | Integer/Long | 分页信息 |
| data.list.scriptNo | Integer | 脚本编号 |
| data.list.scriptName | String | 脚本名称 |
| data.list.scriptTags | JSONArray | 脚本标签 |
| data.list.webReportDevice | JSONObject | 设备信息 |
| data.list.execStatus | Integer | 执行状态 |
| data.list.timeConsuming | Long | 耗时 |
| data.list.status | Integer | 状态 |
| data.list.errorMsg | String | 错误定位 |
| data.list.errorCode | Integer | 错误代码 |
| data.list.taskId / subTaskId / subSubTaskId | String | 任务/子任务/子子任务id |
| data.list.resultCategory | Integer | 结果分类 |

> 代码出处：`cn.testin.realweb.mvc.controller.ReportController.reTestInfo`

### 批量查询测试报告 `POST /report/report/list`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/report/report/list` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 根据 taskId 批量查询测试报告 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 否 | 项目id |
| taskIds | JSONArray | 是 | 任务id列表（空抛异常） |
| subSubTaskIds | JSONArray | 否 | 子子任务id列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.list | JSONArray | PmReportDetail 列表 |

> 代码出处：`cn.testin.realweb.mvc.controller.ReportController.list`

### 批量查询报告汇总 `POST /report/report/summary`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/report/report/summary` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 根据 taskId 批量查询测试报告汇总 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 否 | 项目id |
| taskIds | JSONArray | 是 | 任务id列表（空抛异常） |
| subSubTaskIds | JSONArray | 否 | 子子任务id列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.list | JSONArray | PmReportDetail 列表 |

> 代码出处：`cn.testin.realweb.mvc.controller.ReportController.reportSummary`

### 更新错误类型 `POST /report/report/modify_error_cause_type`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/report/report/modify_error_cause_type` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 更新报告错误原因类型 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| errorCauseTypeId | Integer | 是 | 错误原因类型id（空返回 0） |
| taskId | String | 是 | 任务id（空返回 0） |
| subTaskId | String | 是 | 子任务id（空返回 0） |
| subSubTaskId | String | 是 | 子子任务id（空返回 0） |
| userId | Integer | 否 | 更新人id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 更新结果 |

> 代码出处：`cn.testin.realweb.mvc.controller.ReportController.modifyErrorCauseType`

---

## 十、V3 透传 — 任务（TaskController）

> `cn.testin.realweb.mvc.controller.TaskController`，`@RequestMapping("/task")`。以下接口路由表未单独列出，按代码路径记录。

### 任务详情 `GET /task/task_detail/{task_id}`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/task/task_detail/{task_id}` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 查询任务详情 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 任务id（路径变量） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.projectId | Integer | 项目id |
| data.taskId | String | 任务id |
| data.userid | Integer | 创建者用户id |
| data.userName | String | 用户名 |
| data.userEmail | String | 用户邮箱 |
| data.bizCode | Integer | 业务编码 |
| data.testType | Integer | 测试类型 |
| data.execStatus | Integer | 执行状态 |
| data.startExecTime | Long | 开始执行时间 |
| data.finishTime | Long | 完成时间 |
| data.testResult | Integer | 测试结果 |
| data.execStandard | String | 执行策略 |
| data.browsers | JSONArray | 浏览器信息 |
| data.pcs | JSONArray | 上位机信息 |
| data.retryNum | Integer | 出错重试次数 |
| data.level | Integer | 测试等级 |
| data.networks | String | 网络信息 |
| data.extendedChannel | String | 扩展渠道 |
| data.updateTime | Long | 更新时间 |

> 代码出处：`cn.testin.realweb.mvc.controller.TaskController.getTaskDetail`

### 发送测试计划结果 `GET /task/tasks/send_plan/{task_id}`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/task/tasks/send_plan/{task_id}` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 发送测试计划结果报告 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 任务id（路径变量） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 固定返回 1 |

> 代码出处：`cn.testin.realweb.mvc.controller.TaskController.sendTestPlanResult`

---

## 十一、V3 透传 — 测试计划（TestPlanController）

> `cn.testin.realweb.mvc.controller.TestPlanController`，`@RequestMapping("/realweb")`。

### 测试计划报告导出 `GET /v3/realweb/plan/excel`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realweb/plan/excel` |
| 鉴权 | needLogin=1 |
| 说明 | 测试计划报告 Excel 导出 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| record_id | Long | 否 | 执行记录id |
| user_id | Integer | 否 | 用户id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| （代码未确认） | — | `testPlanExcelService.reportExcel` 返回 ResponseResult，具体 data 字段见服务实现 |

> 代码出处：`cn.testin.realweb.mvc.controller.TestPlanController.excel`

### 脚本重置任务 `POST /v3/realweb/tasks/script_reset`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realweb/tasks/script_reset` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 脚本重置任务 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 否 | 项目id |
| userId | Integer | 否 | 用户id |
| userName | String | 否 | 用户名 |
| taskId | String | 否 | 任务id |
| taskType | Integer | 是 | 任务类型（1=App, 3=Web, 5=PC；null 抛异常） |
| executeRecordTaskId | Long | 否 | 执行记录任务id |
| subSubTaskId | JSONArray | 否 | 子子任务id列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | String | 新任务id（taskId） |

> 代码出处：`cn.testin.realweb.mvc.controller.TestPlanController.scriptResetTask`

### 条件查询子子任务信息 `POST /v3/realweb/tasks/sub_sub_task_info`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realweb/tasks/sub_sub_task_info` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 条件查询子子任务信息 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| errorCauseMessage | String | 否 | 错误原因信息 |
| taskIds | JSONArray | 否 | 任务id列表 |
| deviceIp | String | 否 | 设备ip |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.list.taskId | String | 任务id |
| data.list.subTaskId | String | 子任务id |
| data.list.subSubTaskId | String | 子子任务id |

> 代码出处：`cn.testin.realweb.mvc.controller.TestPlanController.getSubSubTaskInfoByCondition`

---

## 十二、V3 透传 — 任务模板（TestTemplateController）

> `cn.testin.realweb.mvc.controller.TestTemplateController`，`@RequestMapping("/realweb/template")`。

### 条件查询模板列表 `POST /v3/realweb/template/templates`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realweb/template/templates` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 条件查询任务模板列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 是 | 项目id（PageRequestDTO `@NotNull`） |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 页大小 |
| taskName | String | 否 | 任务名 |
| userIds | JSONArray | 否 | 用户id列表 |
| createStartTime | Long | 否 | 创建时间起始 |
| createEndTime | Long | 否 | 创建时间结束 |
| filterIds | JSONArray | 否 | 需要过滤的id |
| ids | JSONArray | 否 | 需要查询的id |
| plan | Boolean | 否 | 是否测试计划 |
| needContent | Boolean | 否 | 是否需要返回 content |
| needScriptDetail | Boolean | 否 | 是否需要脚本详细信息 |
| needScriptAndDeviceBashInfo | Boolean | 否 | 是否需要脚本和设备基础信息 |
| checkStatus | Boolean | 否 | 是否检查执行状态 |
| businessType | Integer | 否 | 区分 web/pc |
| needDataSourceDetail | Boolean | 否 | 是否需要数据源相关信息 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.page / pageSize / totalPage / totalRow | Integer/Long | 分页信息 |
| data.list.taskId | Integer | 任务id |
| data.list.taskName | String | 任务名称 |
| data.list.userName | String | 创建人 |
| data.list.userId | Integer | 用户id |
| data.list.createTime | Long | 创建时间 |
| data.list.isRelation | Integer | 是否关联 |
| data.list.scriptTotal | Integer | 脚本数量 |
| data.list.deviceTotal | Integer | 设备数量 |
| data.list.content | String | 模板信息（提测用） |
| data.list.scriptNos | JSONArray | 脚本编号列表 |
| data.list.deviceIds | JSONArray | 设备id列表 |
| data.list.jobRule | String | 定时策略 |
| data.list.relations | JSONArray | 关联任务 |

> 代码出处：`cn.testin.realweb.mvc.controller.TestTemplateController.listByCondition`

### 批量删除模板 `DELETE /v3/realweb/template/batch_delete`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | DELETE |
| 路径 | `/v3/realweb/template/batch_delete` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 批量删除任务模板 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| templateIds | JSONArray | 是 | 模板id列表 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data | Integer | 删除数量 |

> 代码出处：`cn.testin.realweb.mvc.controller.TestTemplateController.batchRemove`

### 保存模板 json `POST /v3/realweb/template/save_json`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realweb/template/save_json` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 保存任务模板（json 字符串） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| （请求体为 JSON 字符串） | String | 是 | 模板 json 字符串（`@RequestBody String`） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data | Integer | 保存结果（模板id） |

> 代码出处：`cn.testin.realweb.mvc.controller.TestTemplateController.saveTemplate`

### 复制任务模板 `GET /v3/realweb/template/copy`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realweb/template/copy` |
| 鉴权 | needLogin=1 |
| 说明 | 复制 web/pc 任务模板 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| job_id | Integer | 是 | 任务模板id（`@NotNull`） |
| user_id | Integer | 是 | 用户id（`@NotNull`） |
| user_name | String | 是 | 用户名（`@NotEmpty`） |
| dir_id | Integer | 是 | 目录id（`@NotNull`） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 复制结果 |

> 代码出处：`cn.testin.realweb.mvc.controller.TestTemplateController.copyTemplate`

### 更新模板标签 `POST /v3/realweb/template/update_template_content`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | POST |
| 路径 | `/v3/realweb/template/update_template_content` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 更新模板内容（标签） |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| tagList | JSONArray | 否 | 标签id列表 |
| jobIdList | JSONArray | 是 | 任务模板id列表（空抛异常） |
| editType | Integer | 是 | 编辑类型（TagEditEnum；空抛异常） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | Integer | 更新结果 |

> 代码出处：`cn.testin.realweb.mvc.controller.TestTemplateController.updateSourceTag`

### 查询项目模板id集合 `GET /v3/realweb/template/list_template_id`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | GET |
| 路径 | `/v3/realweb/template/list_template_id` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 获取项目下对应的任务模板id集合及总数 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| project_id | Integer | 是 | 项目id |
| business_type | Integer | 是 | 业务类型 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.totalCount | Integer | 任务模板总数 |
| data.templateIds | JSONArray | 任务模板id集合 |

> 代码出处：`cn.testin.realweb.mvc.controller.TestTemplateController.getQuartzJobId`

---

## 十三、V3 透传 — 定时任务日志（QuartzLogController）

> `cn.testin.realweb.mvc.controller.QuartzLogController`，`@RequestMapping("schedule_log")`。

### 删除定时任务日志 `DELETE /schedule_log/schedule_log/remove_by_task_id`

| 属性 | 值 |
|---|---|
| 转发模式 | 🔵 |
| HTTP 方法 | DELETE |
| 路径 | `/schedule_log/schedule_log/remove_by_task_id` |
| 鉴权 | （代码未确认，路由表未列出） |
| 说明 | 根据 taskId 删除定时任务日志 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_id | String | 是 | 任务id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data | Integer | 删除数量 |

> 代码出处：`cn.testin.realweb.mvc.controller.QuartzLogController.removeByTaskId`

---

## 十四、V3→V1 转换（经 realweb 入口，实际由 controlcenter 处理）

> 以下 V3 URL 使用 `/v3/realweb/` 前缀，但网关 `passThroughType=0` + `special_api_action/op` 将其转换为 V1 action/op，最终由**设备控制中心（real-controlcenter）**的 `client.Client` / `device.Device` / `pc.Pc` 处理，real-web 仓库无对应实现类。

### 桌面设备列表 `GET /v3/realweb/desktops`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟡 |
| HTTP 方法 | GET |
| 路径 | `/v3/realweb/desktops` → `action=client&op=Client.disList` |
| 鉴权 | needLogin=1 |
| 说明 | 桌面设备去重列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目id |
| page / pageSize | Integer | 否 | 分页（其它条件见 controlcenter `Client.disList`） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.list | JSONArray | ClientInfoSource 桌面设备列表（controlcenter 返回） |

> 代码出处：（controlcenter 服务 `client.Client.disList`，real-web 内为代理 `cn.testin.realweb.api.controlcenter.ClientApi.disList`）

### 浏览器设备列表 `GET /v3/realweb/devices`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟡 |
| HTTP 方法 | GET |
| 路径 | `/v3/realweb/devices` → `action=device&op=Device.list` |
| 鉴权 | needLogin=1 |
| 说明 | 浏览器设备列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| （代码未确认） | — | 否 | 见 controlcenter `device.Device.list` |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| （代码未确认） | — | controlcenter 设备列表 |

> 代码出处：（controlcenter 服务 `device.Device.list`，real-web 无对应实现）

### 浏览器（PC）列表 `GET /v3/realweb/webs`

| 属性 | 值 |
|---|---|
| 转发模式 | 🟡 |
| HTTP 方法 | GET |
| 路径 | `/v3/realweb/webs` → `action=pc&op=Pc.list` |
| 鉴权 | needLogin=1 |
| 说明 | 浏览器（PC）去重列表 |

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业id |
| projectid | Integer | 是 | 项目id |
| page / pageSize | Integer | 否 | 分页（其它条件见 controlcenter `Pc.list`/`Pc.disList`） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| data.list | JSONArray | PcInfoSource 浏览器列表（controlcenter 返回） |

> 代码出处：（controlcenter 服务 `pc.Pc.list`，real-web 内为代理 `cn.testin.realweb.api.controlcenter.PcApi.disList`）

---

## 附录：内部代理类（非网关对外 HTTP 接口）

> 模块索引「其他ApiServlet」中还记录了 `cn.testin.realweb.api.*` 包下的大量内部代理类。这些类通过 `ApiUtil.doPress` 转发到其它服务（realcfg / controlcenter / notice / scheduling / filesystem 等），**不是网关直接对外的 HTTP 接口**，故不逐接口展开。如需了解其方法，参见 `工程模块清单/web-pc处理服务/07-开放接口文档/其他ApiServlet/` 下的类文档。

| 类 | 目标服务前缀 | 主要方法 |
|---|---|---|
| BizConfigApi | RealCfg | get / list（`cfg.BizConfig.get/list`） |
| CfgCodingApi | RealCfg | 错误码对照查询 |
| DBConfigApi | RealCfg | 数据库环境配置查询 |
| EnvConfigApi | RealCfg | 环境配置查询 |
| TimeoutApi | RealCfg | 超时配置查询 |
| OemSystemApi | RealCfg | OEM 系统参数查询 |
| ErrorCauseTypeV3Api / ErrorCauseOperateLogV3Api | RealCfg | 错误原因类型/操作日志（V3） |
| ClientApi | ControlCenter | disList（`client.Client.disList`） |
| PcApi | ControlCenter | disList（`pc.Pc.disList`） |
| DeviceV3Api | ControlCenter / RealScheduling | getDeviceInfo（V3） |
| PreciseTestApi | CICC | 精准测试覆盖率 |
| CallbackApi | NoticeManager | taskfinish（`http.HttpTask.add`） |
| NoticeApi / EmailTempletApi / SmsTaskApi | NoticeManager | 邮件/消息/短信代理 |
| NoticeReportApi | RealCross | 跨端任务中心状态上报 |
| ChannelEventV3Api | NoticeManager | 通知事件配置查询（V3） |
| PortalApi / PortalTaskApi | RealPortal | 门户任务上报与查询 |
| ProjectApi / UserApi | UserManager | 项目/用户信息查询代理 |
| ScriptApi / ScriptGroupApi / ScriptGroupOperateApi / ScriptGroupV3Api / ParameterSourceApi | Script | 脚本/脚本组/参数数据源代理 |
| UploadApi | Script | 文件上传 |
| ReportApi | RealWeb | 报告 url 与分享 |
| TaskApi / RealTestApi / RealWebApi / McPcTaskApi / TaskV3Api | RealScheduling / RealTest / RealWeb | 任务调度初始化/取消/查询代理 |
| PlanRecordApi / PlanRecordTaskApi / TestPlanApi / TestPlanV3Api | RealTest | 测试计划执行记录代理 |
| DirQuartzApi | RealCfg | 目录与任务模板关联管理 |
| CommonApi | — | 任务模板保存（本地封装） |
