# service-RealTestApi — App 定时任务调度代理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/task/RealTestApi.java`（@Component，继承 `AbstractApi`）
> 类型：远端代理（→ RealTest 服务）
> 转发方式：V1 ApiServlet 前缀 `RealTest`，统一 `action=app`，op 为 `ScheduledJob.*`

## 方法列表

### 1. getQuartzJobInfo — 获取 App 定时任务信息

```java
public ApiJsonResponse getQuartzJobInfo(Map<String, Object> params) throws GeneralException
```

**转发目标**：`op=ScheduledJob.get`，data 含 `projectid/jobId`。

**请求参数**（`Map<String,Object> params`）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| params.projectid | Object | 是 | 项目 id |
| params.jobId | Object | 是 | 定时任务 id |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | ApiJsonResponse | 完整 V1 响应（`{code, msg, data/result/list/objInfo...}`），调用方自行取数据 |

**调用者**：`AppQuartz.java / 412` — 执行前读取 job 配置。

### 2. execute — 执行 App 定时任务

```java
public String execute(Map<String, Object> params) throws GeneralException
```

**转发目标**：`op=ScheduledJob.execute`，data 含 `projectid/jobId`。

**请求参数**（`Map<String,Object> params`）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| params.projectid | Object | 是 | 项目 id |
| params.jobId | Object | 是 | 定时任务 id |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | String | 任务 id（`result` 强转 String） |

**调用者**：`AppQuartz.java`。

### 3. getTaskIdProdByQuartzJob — 分页查询 job 关联任务 id

```java
public List<String> getTaskIdProdByQuartzJob(Integer eid, Integer projectid, Integer jobId, Integer pageNo, Integer pageSize)
```

**转发目标**：`op=ScheduledJob.listTaskIdByJobId`，data 含 `eid/projectid/jobId/pageNo/pageSize`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业 id，null/<=0 抛 `paraInvalid` |
| projectid | Integer | 是 | 项目 id，null/<1 抛 `paraInvalid` |
| jobId | Integer | 是 | 定时任务 id，null/<1 抛 `paraInvalid` |
| pageNo | Integer | 否 | 页码 |
| pageSize | Integer | 否 | 每页大小 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | String | 任务 id 列表 |

**调用者**：`AppQuartz.java` — 重测时取任务 id 列表。

### 4. getlistAllTaskIdByJobId — 查询 job 全部任务 id

```java
public List<String> getlistAllTaskIdByJobId(Integer jobId) throws GeneralException
```

**转发目标**：`op=ScheduledJob.listAllTaskIdByJobId`，data 仅含 `jobId`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| jobId | Integer | 是 | 定时任务 id，null/<1 抛 `paraInvalid` |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | String | 任务 id 列表 |

**调用者**：`AppQuartz.java` — 与上一方法配合统计总数。

### 5. quartzJobList — App 定时任务分页列表

```java
public PageUtils quartzJobList(JSONObject req) throws GeneralException
```

**转发目标**：`op=ScheduledJob.list`，data 支持 `projectid/bizCode/page/pageSize/appVersion/taskDesc/userName/syspfId/suiteId/appName/appId/channelId/orderByCol/orderByType`。

**说明**：返回前给每条记录补 `jobContent`（←taskContent）、`jobDesc`（←taskDesc）两字段，与 Web 端字段对齐；结果反序列化为 `List<QuartzInfo>` 并回填分页属性。

**请求参数**（`JSONObject req`，全部 opt 读取，均可选）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 否 | 项目 id |
| bizCode | Integer | 否 | 业务编码 |
| page / pageSize | Integer | 否 | 分页 |
| appVersion | String | 否 | 应用版本 |
| taskDesc | String | 否 | 任务描述 |
| userName | String | 否 | 用户名 |
| syspfId | Integer | 否 | 系统平台 id |
| suiteId | Integer | 否 | 套件 id |
| appName | String | 否 | 应用名 |
| appId | Integer | 否 | 应用 id |
| channelId | String | 否 | 渠道 id |
| orderByCol / orderByType | String | 否 | 排序字段/方式 |

**返回参数**：`PageUtils`

| 字段 | 类型 | 说明 |
|------|------|------|
| list | List&lt;QuartzInfo&gt; | App 定时任务列表（补 jobContent/jobDesc 字段） |
| currPage | Integer | 当前页 |
| pageSize | Integer | 每页大小 |
| totalPage | Integer | 总页数 |
| totalCount | Integer | 总行数 |

**调用者**：`AppQuartz.java`。

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealTest](../../../app处理服务（real-test）/07-开放接口文档/00-模块索引.md)
