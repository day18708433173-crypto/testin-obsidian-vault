# service-report-Report — 报告页数据查询（概况/步骤/性能/列表）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/service/report/Report.java`（继承 `GenericBaseService`）
> 类型：ApiServlet 本地路由服务类
> 路由方式：action=report, op=Report.<方法>
> 本地注入：`ITaskSummaryService`, `IReportService`, `ITaskDetailService`, `ReportApi`

## 方法列表

### 1. taskSummary — 脚本执行概况

```java
public String taskSummary(ApiRequest apirequest) throws Exception
```

**流程**：
1. skey 经 `ReportApi.getTaskIdByShareId` 换算 taskid，或直接取 taskid，校验非空
2. `ITaskSummaryService.taskSummary(taskid)` 查询 PmTaskSummary
3. 以 fastjson 序列化后放入 RES_OBJECT 返回（null 返回空对象）

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.objInfo | JSONObject | PmTaskSummary 脚本执行概况（null 时返回空对象） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmTaskSummary | 读 |

### 2. scriptSteps — 左侧菜单树（脚本步骤）

```java
public String scriptSteps(ApiRequest apirequest) throws Exception
```

**流程**：校验 taskid、subtaskid（必填），subsubtaskid 可选；组装 conditionMap，`IReportService.scriptSteps(conditionMap)` 返回脚本树详情。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| subtaskid | String | 是 | 子任务ID |
| subsubtaskid | String | 否 | 子子任务ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.objInfo | JSONObject | 脚本步骤树详情（Map） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmReportDetail | 读 |

### 3. stepdetail — 脚本步骤详情

```java
public String stepdetail(ApiRequest apirequest) throws Exception
```

**流程**：校验 taskid/subtaskid/scriptTag/stepid 必填（subsubtaskid 可选）；`IReportService.stepdetail(conditionMap)` 从报告中定位对应步骤返回。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| subtaskid | String | 是 | 子任务ID |
| subsubtaskid | String | 否 | 子子任务ID |
| scriptTag | String | 是 | 脚本标签 |
| stepid | Integer | 是 | 步骤ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.objInfo | JSONObject | 脚本步骤详情（Map） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmReportDetail | 读 |

### 4. scriptCheckInfos — 脚本业务检查点

```java
public String scriptCheckInfos(ApiRequest apirequest) throws Exception
```

**流程**：校验 taskid/subtaskid/subsubtaskid 三级 ID 必填；`IReportService.scriptCheckInfos(conditionMap)` 返回 `Map<String, List<ScriptCheckInfo>>`。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| subtaskid | String | 是 | 子任务ID |
| subsubtaskid | String | 是 | 子子任务ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.objInfo | JSONObject | Map&lt;String, List&lt;ScriptCheckInfo&gt;&gt; 业务检查点 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmReportDetail | 读 |

### 5. scriptRunInfoSummary — 单机页脚本执行概况

```java
public String scriptRunInfoSummary(ApiRequest apirequest) throws Exception
```

**流程**：校验 taskid、subtaskid（subsubtaskid 可选）；`IReportService.scriptRunInfoSummary(conditionMap)` 返回单机执行概况。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| subtaskid | String | 是 | 子任务ID |
| subsubtaskid | String | 否 | 子子任务ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.objInfo | JSONObject | 单机页脚本执行概况（Map） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmReportDetail / PmScriptRunInfo | 读 |

### 6. performanceCondition — 网络性能数据查询条件

```java
public String performanceCondition(ApiRequest apirequest) throws Exception
```

**流程**：校验 taskid/subtaskid/subsubtaskid/scriptTag/stepid 全部必填；`IReportService.performanceCondition(...)` 返回可筛选条件。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| subtaskid | String | 是 | 子任务ID |
| subsubtaskid | String | 是 | 子子任务ID |
| scriptTag | String | 是 | 脚本标签 |
| stepid | Integer | 是 | 步骤ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.objInfo | JSONObject | 网络性能数据可筛选条件（Map） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmReportDetail | 读 |

### 7. stepInternetInfo — 单步骤网络信息解析

```java
public String stepInternetInfo(ApiRequest apirequest) throws Exception
```

**流程**：
1. 校验 taskid/subtaskid/subsubtaskid/scriptTag/stepid 全部必填
2. 可选过滤：initiatorTypes（资源类型）、statusCodes（状态码）、requestUrl
3. `IReportService.stepInternetInfo(...)` 返回 `List<NetPerformance>`，经父类 `listToResList` 转 JSON 数组返回

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| subtaskid | String | 是 | 子任务ID |
| subsubtaskid | String | 是 | 子子任务ID |
| scriptTag | String | 是 | 脚本标签 |
| stepid | Integer | 是 | 步骤ID |
| initiatorTypes | JSONArray | 否 | 资源类型过滤 |
| statusCodes | JSONArray | 否 | 状态码过滤 |
| requestUrl | String | 否 | 请求地址过滤 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.list | JSONArray | List&lt;NetPerformance&gt; 单步骤网络信息列表 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmReportDetail | 读 |

### 8. getBrowserInfo — 报告详情获取浏览器信息

```java
public String getBrowserInfo(ApiRequest apirequest) throws Exception
```

**流程**：校验 taskid/subtaskid；`IReportService.getBrowserInfo(taskid, subtaskid)` 返回 BrowserInfo。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| subtaskid | String | 是 | 子任务ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.objInfo | JSONObject | BrowserInfo 浏览器信息 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmDeviceRunInfo（浏览器维度） | 读 |

### 9. getClientInfo — 报告详情获取 PC 客户端信息

```java
public String getClientInfo(ApiRequest apirequest) throws Exception
```

**流程**：校验 taskid/subtaskid；`IReportService.getClientInfo(taskid, subtaskid)` 返回 ClientInfo。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| subtaskid | String | 是 | 子任务ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.objInfo | JSONObject | ClientInfo PC 客户端信息 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmDeviceRunInfo（PC 维度） | 读 |

### 10. testProcess — 测试过程

```java
public String testProcess(ApiRequest apirequest) throws Exception
```

**流程**：校验 taskid/subsubtaskid/name/stage 必填；`IReportService.testProcess(taskid, subsubtaskid, name, stage)` 返回过程列表（RES_LIST）。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| subsubtaskid | String | 是 | 子子任务ID |
| name | String | 是 | 过程名称 |
| stage | String | 是 | 阶段 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.list | JSONArray | List&lt;TestProcess&gt; 测试过程列表 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmReportDetail（TestProcess 内嵌） | 读 |

### 11. url — 获取报告 url

```java
public String url(ApiRequest apirequest) throws Exception
```

**流程**：
1. skey/taskid 换算与校验，projectid 必填
2. 按 taskid 前缀拼接报告地址：wt→`Config.REPORT_WEB_URL`、pt→`Config.REPORT_PC_URL`，带 `?taskid=` 参数
3. 返回 url 字符串

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| projectid | Integer | 是 | 项目组ID |
| subtaskid | String | 否 | 子任务ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | String | 报告 url |

### 12. list — 报告详情信息（分页）

```java
public String list(ApiRequest apirequest) throws Exception
```

**流程**：
1. skey/taskid 换算与校验；解析 needTestCases、subtaskid、resultCategorys（大于 `USER_MIN_ERROR_CODE` 拆分为 errorCauseTypeIds）、errorCauseTypeIds、keywords、subsubTaskIds
2. 分页默认 page=1、pageSize=200（上限 200），排序 `orderNum`
3. `ITaskDetailService.get(taskid, null)` 读取任务；needTestCases=0 且非恒生任务组时 excludes 掉 `testCases` 字段（防 OOM）
4. `IReportService.baseList(conditionMap, page, pageSize)` 分页查询 PmReportDetail，逐条 Gson 序列化返回，附带分页信息

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| needTestCases | Integer | 否 | 是否返回 testCases 字段（0 时非恒生任务组排除该字段） |
| subtaskid | String | 否 | 子任务ID |
| resultCategorys | String | 否 | 结果分类列表（>USER_MIN_ERROR_CODE 拆为 errorCauseTypeIds） |
| errorCauseTypeIds | String | 否 | 错误原因类型ID列表 |
| keywords | JSONArray | 否 | 关键字过滤 |
| subsubTaskIds | JSONArray | 否 | 子子任务ID过滤 |
| page | Integer | 否 | 页码（默认 1） |
| pageSize | Integer | 否 | 每页大小（默认 200，上限 200） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.list | JSONArray | PmReportDetail 列表 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmTaskDetail | 读 |
| MongoDB | PmReportDetail | 读 |

### 13. modifyResultCategory — 更改子子任务结果分类

```java
public String modifyResultCategory(ApiRequest apirequest) throws Exception
```

**流程**：
1. 校验 taskid/subtaskid 必填，resultCategory 必填且能映射 `ResultCategoryEnum`
2. subsubtaskid 非空时为脚本级操作
3. `IReportService.modifyResultCategory(taskid, subtaskid, subsubtaskid, resultCategory, userid)` 更新，返回 1/0

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是 | 任务ID |
| subtaskid | String | 是 | 子任务ID |
| subsubtaskid | String | 否 | 子子任务ID（非空时脚本级操作） |
| resultCategory | Integer | 是 | 结果分类（须能映射 `ResultCategoryEnum`） |
| userid | Integer | 否 | 操作用户ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 1 成功 / 0 失败 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmReportDetail | 写 |

## 相关文档

- [00-分支索引](00-分支索引.md)
- [service-task-Task](service-task-Task.md)（任务侧详情/执行信息）
- [service-report-Excel](service-report-Excel.md) / [service-report-Pdf](service-report-Pdf.md)（报告导出）
- [service-quartz-Report](service-quartz-Report.md)（定时任务侧报告 url）
