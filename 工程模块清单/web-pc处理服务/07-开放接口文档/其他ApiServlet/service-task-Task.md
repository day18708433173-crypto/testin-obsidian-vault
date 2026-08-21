# service-task-Task — Web/PC 测试任务的创建、控制与详情查询

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/service/task/Task.java`（继承 `GenericBaseService`）
> 类型：ApiServlet 本地路由服务类
> 路由方式：action=task（scheduling 等别名）, op=Task.<方法>
> 本地注入：`ITaskService`, `ITaskDetailService`, `IScriptRunInfoService`, `IDeviceRunInfoService`, `IReportService`, `ReportApi`，另直接引用 `ParameterSourceApi`、`TestPlanV3Api`（SpringUtil 获取）
> 返回结构：V1 统一 `{code, msg, data}`，code=0 成功；data 子键按语义为 `result`（标量）、`objInfo`（对象）、`list`/`page`/`pageSize`/`totalRow`/`totalPage`（列表与分页）。

## 方法列表

### 1. create — 新增 web/PC 测试任务

```java
public String create(ApiRequest apirequest) throws Exception
```

**用途**：创建 Web/PC 端测试任务，返回 taskid。

**流程**：
1. `checkCreateParams` 校验必填参数（eid/projectid/userid/bizCode/taskDescr/browsers 或 pcs/scripts）
2. 若有 apikey 则写入 `apkey`
3. 调用 `ITaskService.create(reqJson)` 本地创建任务，得到 taskid
4. 捕获 `GeneralException` 返回对应错误码，否则返回 `result=taskid`

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业ID（>0） |
| projectid | Integer | 是 | 项目组ID（>0） |
| userid | Integer | 是 | 用户ID（>0） |
| bizCode | Integer | 是 | 业务编码（非空） |
| taskDescr | String | 是 | 任务描述（非空） |
| browsers | JSONArray | 是(与pcs二选一) | 浏览器列表，与 pcs 至少一个非空 |
| pcs | JSONArray | 是(与browsers二选一) | PC 列表，与 browsers 至少一个非空 |
| scripts | JSONArray | 是 | 脚本列表（长度≥1） |
| apkey | String | 否 | API key（由请求头 apikey 自动写入） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | String | 新任务ID（taskid） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmTaskDetail | 写（→ITaskService.create） |

### 2. cancel — 终止测试

```java
public String cancel(ApiRequest apirequest) throws Exception
```

**用途**：按 taskid（可带 subtaskid / 恒生 taskGroup）终止任务。

**流程**：
1. 校验 projectid、taskid 非空
2. 若带 `taskGroup` 则校验其 `id` 非空
3. `ITaskService.cancel(taskid, subtaskid, taskGroup)` 执行终止，返回影响数

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| taskid | String | 是 | 任务ID |
| subtaskid | String | 否 | 子任务ID |
| taskGroup | JSONObject | 否 | 恒生任务组，存在时其内 id 必填 |
| taskGroup.id | String | 是(条件) | taskGroup 存在时必填 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 终止影响数 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmTaskDetail | 写（→ITaskService.cancel） |

### 3. batchCancel — 批量终止测试

```java
public String batchCancel(ApiRequest apirequest) throws Exception
```

**流程**：
1. 校验 projectid、taskids 数组非空，过滤空白项
2. 逐个调用 `ITaskService.cancel(taskid, null, taskGroup)`，成功数累加
3. 单个失败仅记录日志不中断，返回累计成功数

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| taskids | JSONArray | 是 | 任务ID数组（长度≥1） |
| taskGroup | JSONObject | 否 | 恒生任务组，存在时其内 id 必填 |
| taskGroup.id | String | 是(条件) | taskGroup 存在时必填 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 终止成功条数（累计） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmTaskDetail | 写（→ITaskService.cancel） |

### 4. scriptRunInfos — 脚本执行详情（分页）

```java
public String scriptRunInfos(ApiRequest apirequest) throws Exception
```

**流程**：
1. 支持 `skey` 经 `ReportApi.getTaskIdByShareId` 换算 taskid；校验 taskid
2. 解析过滤条件：resultCategorys（大于 `USER_MIN_ERROR_CODE` 的拆分为 errorCauseTypeIds）、errorCauseTypeIds、subtaskids、scriptDescr/scriptNo/errorMsg/type/version、系统信息五元组（systemBitness/cpuArch/systemVersion/systemName/systemType）
3. 分页参数默认 page=1、pageSize=100（上限 100）
4. 组装 conditionMap（含 `retestMark=false`），按 taskid 前缀排序：pt→`pcScript.orderNum`、wt→`webScript.orderNum`
5. `IScriptRunInfoService.scriptRunInfos` 查询；有 `customizeErrorMsg` 时覆盖 errorMsg

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| resultCategorys | String | 否 | 结果分类列表（逗号分隔，>USER_MIN_ERROR_CODE 的拆为 errorCauseTypeIds） |
| errorCauseTypeIds | String | 否 | 错误原因类型ID列表（逗号分隔） |
| subtaskids | JSONArray | 否 | 子任务ID列表 |
| scriptDescr | String | 否 | 脚本描述 |
| scriptNo | Integer | 否 | 脚本编号 |
| errorMsg | String | 否 | 错误信息 |
| type | String | 否 | 类型 |
| version | String | 否 | 版本 |
| systemBitness | String | 否 | 系统位数 |
| cpuArch | String | 否 | CPU架构 |
| systemVersion | String | 否 | 操作系统版本 |
| systemName | String | 否 | 操作系统名称 |
| systemType | String | 否 | 操作系统类型 |
| page | Integer | 否 | 页码（默认 1） |
| pageSize | Integer | 否 | 每页大小（默认 100，上限 100） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.objInfo | JSONObject | 脚本执行结果集合（含 scriptRunInfos BaseList 及其分页信息 total 等） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmScriptRunInfo | 读 |

### 5. browserRunInfos — 浏览器执行详情（分页）

```java
public String browserRunInfos(ApiRequest apirequest) throws Exception
```

**流程**：
1. skey/taskid 换算与校验
2. 解析 browserTypes（兼容拼错的 `bowserTypes`）、ips、osNames、versions、resultCategorys（>100 拆分为 errorCauseTypes）
3. 分页默认 page=1、pageSize=100；排序 `createtime DESC`，`retestMark=false`
4. `IDeviceRunInfoService.browserRunInfos` 查询返回

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| browserTypes | JSONArray | 否 | 浏览器类型（兼容拼写 bowserTypes） |
| bowserTypes | JSONArray | 否 | 浏览器类型（拼写兼容别名） |
| ips | JSONArray | 否 | IP 列表 |
| osNames | JSONArray | 否 | 操作系统名称列表 |
| resultCategorys | JSONArray | 否 | 结果分类（>100 拆为 errorCauseTypes） |
| versions | JSONArray | 否 | 版本列表 |
| page | Integer | 否 | 页码（默认 1） |
| pageSize | Integer | 否 | 每页大小（默认 100，上限 100） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.objInfo | JSONObject | 浏览器执行结果集合（含分页信息） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmDeviceRunInfo（浏览器维度） | 读 |

### 6. clientRunInfos — PC 执行详情（分页）

```java
public String clientRunInfos(ApiRequest apirequest) throws Exception
```

**流程**：
1. skey/taskid 换算与校验
2. 解析 systemVersions/systemNames/systemTypes/ips、resultCategorys（>100 拆分为 errorCauseTypes）
3. 分页与排序同 browserRunInfos
4. `IDeviceRunInfoService.clientRunInfos` 查询返回

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| systemVersions | JSONArray | 否 | 系统版本列表 |
| systemNames | JSONArray | 否 | 系统名称列表 |
| systemTypes | JSONArray | 否 | 系统类型列表 |
| ips | JSONArray | 否 | IP 列表 |
| resultCategorys | JSONArray | 否 | 结果分类（>100 拆为 errorCauseTypes） |
| page | Integer | 否 | 页码（默认 1） |
| pageSize | Integer | 否 | 每页大小（默认 100，上限 100） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.objInfo | JSONObject | PC 客户端执行结果集合（含分页信息） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmDeviceRunInfo（PC 客户端维度） | 读 |

### 7. ignore — 忽略（本期未实现）

```java
public String ignore(ApiRequest apirequest) throws Exception
```

**用途**：按 subsubtaskid 是否存在分别调用 `IScriptRunInfoService.ignore` / `IDeviceRunInfoService.ignore`；注释标注 TODO 本期暂不做，方法返回 null。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是 | 任务ID |
| subtaskid | String | 是 | 子任务ID |
| subsubtaskid | String | 否 | 子子任务ID |

**返回参数**：返回 null（本期未实现）。

### 8. internetInfo — 获取网络信息（未实现）

```java
public String internetInfo(ApiRequest apirequest) throws Exception
```

仅解析 taskid/subsubtaskid，方法体为 TODO，返回 null。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 否 | 任务ID |
| subsubtaskid | String | 否 | 子子任务ID |

**返回参数**：返回 null（未实现）。

### 9. verification — 校验 taskid / skey 有效性

```java
public String verification(ApiRequest apirequest) throws Exception
```

**流程**：
1. 调用父类 `taskInfoSupplement(skey, taskid, userid, eid, userprojectids)` 校验并补充任务信息（会读 PmTaskDetail 并做 projectid 归属校验）
2. taskid 为空返回参数错误，否则返回 `result=1`

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| userid | Integer | 否 | 用户ID（归属校验） |
| eid | Integer | 否 | 企业ID（归属校验） |
| userprojectids | JSONArray | 否 | 用户项目ID集合（归属校验） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 固定 1 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmTaskDetail | 读 |

### 10. detail — 任务详情

```java
public String detail(ApiRequest apirequest) throws Exception
```

**流程**：
1. skey/taskid 换算与校验，支持 `keywords` 限定返回字段
2. `ITaskDetailService.get(taskid, keywords)` 读 PmTaskDetail，未命中抛 noneData
3. `filterDevice` 过滤补测设备（retestMark=retest 的浏览器/PC）
4. `ParameterSourceApi.getTagInfoList` 补充数据源标签/skip 标签名称
5. 有 `scriptStatuses` 时：`IReportService.getPmReportDetailByTaskIdAndScriptStatus` 查询，按 rawDataUuid/orderNum 调 `removeNoNeedScripts` 裁剪 scripts/rawScripts
6. 计算 scriptNum：DATA/RETENTION/REPLACE 标准走 `IScriptRunInfoService.scriptRunInfos` total；否则 scripts 数 × browsers 或 pcs 数
7. `TestPlanV3Api.recordRelation` 补充计划执行记录关联 relations

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| keywords | JSONArray | 否 | 限定返回字段列表 |
| scriptStatuses | JSONArray | 否 | 脚本执行状态过滤（触发脚本裁剪） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.objInfo | JSONObject | PmTaskDetail 任务详情（含 sourceTagName/skipTagName/scriptNum/relations 等） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmTaskDetail | 读 |
| MongoDB | PmReportDetail | 读 |
| MongoDB | PmScriptRunInfo | 读 |
| Remote DataSource | 标签表 | 读 |

### 11. runInfoConditions — 执行详情筛选条件

```java
public String runInfoConditions(ApiRequest apirequest) throws Exception
```

**流程**：校验 taskid、type；type=script 走 `IScriptRunInfoService.runInfoConditions`，否则走 `IDeviceRunInfoService.runInfoConditions`，返回可筛选条件集合。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| type | String | 是 | 类型，`script` 走脚本条件，其它走设备条件 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.objInfo | JSONObject | 可筛选条件集合 |

### 12. modify — 修改任务详情（报告总结）

```java
public String modify(ApiRequest apirequest) throws GeneralException
```

**流程**：校验 taskid；构造仅含 taskid+summarize 的 PmTaskDetail，`ITaskDetailService.modifyTaskDetail(taskDetail, false)` 更新，返回 1/0。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是 | 任务ID |
| summarize | String | 否 | 报告总结 |

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
| MongoDB | PmTaskDetail | 写 |

### 13. repeatTest — 批量补测（浏览器/脚本）

```java
public String repeatTest(ApiRequest apirequest) throws Exception
```

**流程**：
1. Assert 校验 taskid、subtasks 非空
2. 设备数组：pt 前缀任务取 `pcs`，其余取 `browsers`
3. `subsubtaskinfo` JSON 反序列化为 `Map<String, List<Map<String,String>>>`（脚本级补测信息）
4. `ITaskService.repeatTest(taskid, subtasks, devices, subsubtaskinfo)` 执行，返回 1/0

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是 | 任务ID |
| subtasks | JSONArray | 是 | 子任务ID数组（长度≥1） |
| browsers | JSONArray | 否(pt取pcs) | 补测浏览器列表；pt 前缀任务改用 pcs |
| pcs | JSONArray | 否(pt任务使用) | 补测 PC 列表 |
| subsubtaskinfo | JSONObject | 否 | 脚本级补测信息（Map<String, List<Map>>） |

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
| MongoDB | PmTaskDetail | 写（→ITaskService.repeatTest） |

### 14. modifyErrorMsg — 更新报告错误信息

```java
public String modifyErrorMsg(ApiRequest apirequest) throws Exception
```

**流程**：校验 taskid/subtaskid/subsubtaskid；`IScriptRunInfoService.modifyErrorMsg(taskid, subtaskid, subsubtaskid, errorMsg, userId)` 更新自定义错误信息。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是 | 任务ID |
| subtaskid | String | 是 | 子任务ID |
| subsubtaskid | String | 是 | 子子任务ID |
| errorMsg | String | 否 | 自定义错误信息 |
| userId | Integer | 否 | 操作用户ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 更新结果 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmScriptRunInfo | 写 |

### 15. sendEmail — 重新发送结果邮件

```java
public String sendEmail(ApiRequest apirequest) throws Exception
```

**流程**：校验 taskid；`ITaskDetailService.sendEmail(taskid)` 触发重发，返回结果码。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是 | 任务ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 结果码 |

### 16. execute — 触发任务执行

```java
public String execute(ApiRequest apirequest) throws Exception
```

**流程**：校验 taskid；`ITaskService.execute(taskid)` 触发执行。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是 | 任务ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 结果码 |

### 17. pause — 暂停任务下发

```java
public String pause(ApiRequest apirequest) throws Exception
```

**流程**：校验 taskid；`ITaskService.pause(taskid)` 暂停下发。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是 | 任务ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 结果码 |

### 18. resume — 恢复任务下发

```java
public String resume(ApiRequest apirequest) throws Exception
```

**流程**：校验 taskid；`ITaskService.resume(taskid)` 恢复下发。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是 | 任务ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 结果码 |

## 公共辅助方法（非路由）

- `removeNoNeedScripts(Map, Set, List<ScriptInfo>, List<ScriptInfo>)` — detail 的脚本裁剪逻辑：按 rawDataUuid/orderNum/groupid 对应关系剔除不在结果集内的 scripts/rawScripts，并重算执行次数（id 后缀 `_count`）
- `filterDevice(PmTaskDetail)` — 移除 retestMark=retest 的浏览器与 PC（补测设备不展示）

## 相关文档

- [00-分支索引](00-分支索引.md)
- [service-McPcTaskApi](service-McPcTaskApi.md)（PC 任务创建对等服务类）
- [service-task-TestResult](service-task-TestResult.md) / [service-task-TestProcess](service-task-TestProcess.md)
- [service-report-Report](service-report-Report.md)
