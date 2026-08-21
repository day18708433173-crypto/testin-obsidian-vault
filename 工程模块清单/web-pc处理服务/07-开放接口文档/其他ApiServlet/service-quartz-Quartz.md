# service-quartz-Quartz — 定时任务增删改查与执行入口

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/service/quartz/Quartz.java`（无父类，SpringContextUtil 手动取 Bean）
> 类型：ApiServlet 本地路由服务类
> 路由方式：action=quartz, op=Quartz.<方法>
> 本地注入：`QuartzJobServiceImpl`, `QuartzJobMapper`；业务分发经 `QuartzFactory.getType(businessType)` → `IQuartz`（app/web/mcpc/cross 实现）

## 方法列表

### 1. add — 新增定时任务

```java
public String add(ApiRequest apiRequest) throws Exception
```

**流程**：
1. `verifyParams` 校验公共参数（eid/projectid/taskDescr/userid/userName/businessType/bizCode）
2. 按 businessType 追加校验：app→scripts；web→scripts+browsers；mcpc→scripts+pcs；cross→scripts
3. `QuartzFactory.getType` 获取对应 `IQuartz` 实现，`quartz.add(reqJson)` 创建
4. 异常分级返回（GeneralException 带业务码 / 其他 execFailed）

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业ID（>0） |
| projectid / projectId | Integer | 是 | 项目组ID（>0） |
| taskDescr / desc | String | 是 | 任务描述（非空） |
| userid / userId | Integer | 是 | 用户ID（>0） |
| userName | String | 是 | 用户名（非空） |
| businessType | Integer | 是 | 业务类型（app/web/mcpc/cross） |
| bizCode | Integer | 是 | 业务编码（>0） |
| scripts | JSONArray | 是(按类型) | 脚本列表（app/web/mcpc/cross 均需） |
| browsers | JSONArray | 是(web) | 浏览器列表（businessType=web 时） |
| pcs | JSONArray | 是(mcpc) | PC 列表（businessType=mcpc 时） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 创建结果 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MySQL db_common | quartz_job | 写（→IQuartz.add） |

### 2. update — 修改定时任务

```java
public String update(ApiRequest apiRequest) throws Exception
```

**流程**：校验 jobId>0；同 add 的公共参数+按类型参数校验（不含 cross）；`IQuartz.update(reqJson)` 更新。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| jobId | Integer | 是 | 定时任务ID（>0） |
| eid | Integer | 是 | 企业ID（>0） |
| projectid / projectId | Integer | 是 | 项目组ID（>0） |
| taskDescr / desc | String | 是 | 任务描述（非空） |
| userid / userId | Integer | 是 | 用户ID（>0） |
| userName | String | 是 | 用户名（非空） |
| businessType | Integer | 是 | 业务类型（app/web/mcpc） |
| bizCode | Integer | 是 | 业务编码（>0） |
| scripts | JSONArray | 是(按类型) | 脚本列表 |
| browsers | JSONArray | 是(web) | 浏览器列表（web） |
| pcs | JSONArray | 是(mcpc) | PC 列表（mcpc） |

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
| MySQL db_common | quartz_job | 写 |

### 3. list — 查询定时任务列表（分页）

```java
public String list(ApiRequest apiRequest) throws GeneralException
```

**流程**：校验 eid/projectid/page/pageSize/businessType；`IQuartz.list(reqJson)` 返回 `PageUtils`（app 类型返回字段在实现内有兼容处理）。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业ID（>0） |
| projectid | Integer | 是 | 项目组ID（>0） |
| page | Integer | 是 | 页码（>0） |
| pageSize | Integer | 是 | 每页大小（>0） |
| businessType | String | 是 | 业务类型 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | JSONObject | PageUtils 分页对象（含 list 及分页字段） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MySQL db_common | quartz_job | 读 |

### 4. delete — 删除定时任务

```java
public String delete(ApiRequest apiRequest) throws Exception
```

**流程**：`authenticationWebParams` 校验 jobId/eid/projectid；校验 businessType；`IQuartz.delete(jobId, eid, projectId, deleteRecords, userId, userName)` 删除（deleteRecords=1 时同删执行记录）。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| jobId | Integer | 是 | 定时任务ID（>0） |
| eid | Integer | 是 | 企业ID（>0） |
| projectid | Integer | 是 | 项目组ID（>0） |
| businessType | String | 是 | 业务类型 |
| userid | Integer | 否 | 用户ID |
| username | String | 否 | 用户名 |
| deleteRecords | Integer | 否 | 是否同时删除执行记录（默认 0） |

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
| MySQL db_common | quartz_job / quartz_job_log | 写 |

### 5. batchDelete — 批量删除定时任务

```java
public String batchDelete(ApiRequest apiRequest) throws Exception
```

**流程**：`authenticationBatchParams` 校验 jobIds/eid/projectid；`IQuartz.delete(jobIds, ...)` 批量删除，返回成功数。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| jobIds | JSONArray | 是 | 定时任务ID数组（非空） |
| eid | Integer | 是 | 企业ID（>0） |
| projectid | Integer | 是 | 项目组ID（>0） |
| businessType | String | 是 | 业务类型 |
| userid | Integer | 否 | 用户ID |
| username | String | 否 | 用户名 |
| deleteRecords | Integer | 否 | 是否同时删除执行记录（默认 0） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 删除成功数 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MySQL db_common | quartz_job / quartz_job_log | 写 |

### 6. stop — 暂停定时任务

```java
public String stop(ApiRequest apiRequest) throws Exception
```

**流程**：校验 jobId/eid/projectid；`QuartzJobServiceImpl.stop(jobId, eid, projectId, userId)` 暂停调度。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| jobId | Integer | 是 | 定时任务ID（>0） |
| eid | Integer | 是 | 企业ID（>0） |
| projectid | Integer | 是 | 项目组ID（>0） |
| userid | Integer | 否 | 用户ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 暂停结果 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MySQL db_common | quartz_job | 写 |

### 7. batchStop — 批量暂停定时任务

```java
public String batchStop(ApiRequest apiRequest) throws Exception
```

**流程**：校验 jobIds/eid/projectid；逐个 `quartzJobService.stop`，单条失败仅记日志，累计成功数返回。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| jobIds | JSONArray | 是 | 定时任务ID数组（非空） |
| eid | Integer | 是 | 企业ID（>0） |
| projectid | Integer | 是 | 项目组ID（>0） |
| userid | Integer | 否 | 用户ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 暂停成功数（累计） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MySQL db_common | quartz_job | 写 |

### 8. reset — 恢复定时任务

```java
public String reset(ApiRequest apiRequest) throws Exception
```

**流程**：校验 jobId/eid/projectid；`quartzJobService.reset(jobId, eid, projectId, userId)` 恢复调度。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| jobId | Integer | 是 | 定时任务ID（>0） |
| eid | Integer | 是 | 企业ID（>0） |
| projectid | Integer | 是 | 项目组ID（>0） |
| userid | Integer | 否 | 用户ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 恢复结果 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MySQL db_common | quartz_job | 写 |

### 9. batchReset — 批量恢复定时任务

```java
public String batchReset(ApiRequest apiRequest) throws Exception
```

**流程**：校验 jobIds/eid/projectid；逐个 `quartzJobService.reset`，累计成功数返回。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| jobIds | JSONArray | 是 | 定时任务ID数组（非空） |
| eid | Integer | 是 | 企业ID（>0） |
| projectid | Integer | 是 | 项目组ID（>0） |
| userid | Integer | 否 | 用户ID |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 恢复成功数（累计） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MySQL db_common | quartz_job | 写 |

### 10. start — 立即执行定时任务

```java
public String start(ApiRequest apiRequest) throws Exception
```

**流程**：
1. 校验 jobId/eid/projectid/businessType
2. 组装 map（jobId、extendedChannel、extendedChannelUrl、eid/projectid/userid/username、`isManualExecution=1`、checkDeviceStatus）
3. `IQuartz.execute(map)` 触发立即执行，返回 taskid 等信息

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| jobId | String | 是 | 定时任务ID |
| eid | Integer | 是 | 企业ID（>0） |
| projectid | Integer | 是 | 项目组ID（>0） |
| businessType | Integer | 是 | 业务类型（>0） |
| extendedChannel | String | 否 | 扩展渠道 |
| extendedChannelUrl | String | 否 | 扩展渠道 url |
| userid | Integer | 否 | 用户ID |
| username | String | 否 | 用户名 |
| checkDeviceStatus | Integer | 否 | 是否检查设备状态（默认 0） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | String | 执行结果（taskid 等信息） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MySQL db_common | quartz_job_log | 写（执行记录） |
| MongoDB | PmTaskDetail | 写（→IQuartz.execute 创建任务） |

### 11. batchStart — 批量立即执行定时任务

```java
public String batchStart(ApiRequest apiRequest) throws Exception
```

**流程**：同 start，遍历 jobIds 逐个 `quartz.execute(map)`，收集非空结果列表返回；单条失败仅记日志。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| jobIds | JSONArray | 是 | 定时任务ID数组（非空） |
| eid | Integer | 是 | 企业ID（>0） |
| projectid | Integer | 是 | 项目组ID（>0） |
| businessType | Integer | 是 | 业务类型（>0） |
| extendedChannel | String | 否 | 扩展渠道 |
| extendedChannelUrl | String | 否 | 扩展渠道 url |
| userid | Integer | 否 | 用户ID |
| username | String | 否 | 用户名 |
| checkDeviceStatus | Integer | 否 | 是否检查设备状态（默认 0） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | JSONArray | List&lt;String&gt; 执行结果列表 |

### 12. quartzJobParams — 修改时回显数据

```java
public String quartzJobParams(ApiRequest apiRequest) throws Exception
```

**流程**：校验 jobId/eid/projectid；`IQuartz.QuartzJobParams(reqJson)` 查询任务详情回显，空结果返回 execSqlFailed。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| jobId | Integer | 是 | 定时任务ID（>0） |
| eid | Integer | 是 | 企业ID（>0） |
| projectid | Integer | 是 | 项目组ID（>0） |
| businessType | String | 是 | 业务类型（用于 QuartzFactory 分发） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | JSONObject | 定时任务详情（Map） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MySQL db_common | quartz_job | 读 |

### 13. listRealTask — 定时任务关联的真实任务列表

```java
public String listRealTask(ApiRequest apiRequest) throws GeneralException
```

**流程**：校验 jobId/projectid/page/pageSize/businessType 均存在；`IQuartz.listRealTask(reqJson)` 分页返回执行记录。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| jobId | String | 是 | 定时任务ID |
| projectid | Integer | 是 | 项目组ID |
| page | Integer | 是 | 页码 |
| pageSize | Integer | 是 | 每页大小 |
| businessType | String | 是 | 业务类型 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | JSONObject | PageUtils 分页对象（执行记录列表） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MySQL db_common | quartz_job_log | 读 |

### 14. conditionalQuery — 条件查询 app 信息

```java
public String conditionalQuery(ApiRequest apiRequest)
```

**流程**：校验 page/pageSize/eid/projectid；`IQuartz.conditionalQuery(reqJson)` 分页查询。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | Integer | 是 | 页码（>0） |
| pageSize | Integer | 是 | 每页大小（>0） |
| eid | Integer | 是 | 企业ID（>0） |
| projectid | Integer | 是 | 项目组ID（>0） |
| businessType | String | 是 | 业务类型（用于 QuartzFactory 分发） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | JSONObject | PageUtils 分页对象 |

### 15. retest — 重测

```java
public String retest(ApiRequest apiRequest)
```

**流程**：校验 eid/projectid/taskId/businessType；`IQuartz.retest(reqJson)` 执行重测，返回结果 JSON。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业ID（>0） |
| projectid | Integer | 是 | 项目组ID（>0） |
| taskId | String | 是 | 任务ID |
| businessType | Integer | 是 | 业务类型（>0） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | JSONObject | 重测结果 JSON |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmTaskDetail | 写（→IQuartz.retest） |

### 16. createScriptGroup — 提测脚本合并为脚本组

```java
public String createScriptGroup(ApiRequest apiRequest)
```

**流程**：`verifyScriptGroupParams` 校验 businessType/scripts/groupDesc/scriptType/userId/projectId/userName；`IQuartz.createScriptGroup(reqJson)` 创建，result<=0 返回 execFailed。

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| businessType | Integer | 是 | 业务类型（>0） |
| scripts | JSONArray | 是 | 脚本列表 |
| groupDesc | String | 是 | 脚本组描述（非空） |
| scriptType | Integer | 是 | 脚本类型（>0） |
| userId | Integer | 是 | 用户ID（>0） |
| projectId | Integer | 是 | 项目组ID（>0） |
| userName | String | 是 | 用户名（非空） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功（result<=0 时 execFailed） |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 脚本组创建结果 |

### 17. getScriptGroupScript — 获取定时任务引用的脚本 ID 集合

```java
public String getScriptGroupScript(ApiRequest apiRequest)
```

**流程**：
1. `QuartzJobMapper.selectList` 查 delete_status=0 的 job_content/biz_code
2. 过滤 bizCode 为 4100/4200/4300 的记录，解析 jobContent.scripts 数组
3. 收集去重 scriptid，返回列表

**请求参数**（reqJson）：无。

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.list | JSONArray | List&lt;String&gt; 定时任务引用的脚本 ID 集合 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MySQL db_common | quartz_job | 读 |

## 相关文档

- [00-分支索引](00-分支索引.md)
- [service-McPcTaskApi](service-McPcTaskApi.md)（Quartz 执行链路创建任务）
- [service-quartz-Report](service-quartz-Report.md)
- [service-task-Task](service-task-Task.md)
