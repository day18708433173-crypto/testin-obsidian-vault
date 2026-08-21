# service-ReportApi — 报告 URL 与分享

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/report/ReportApi.java`（@Component("reportApi")）
> 类型：远端代理（→ RealWeb / RealTest 服务）
> 转发方式：V1 ApiServlet 前缀（`ApiUtil.doPress`，web 端走 "RealWeb" 前缀，app 端走 "RealTest" 前缀）

## 方法列表

### 1. getWebReportUrl — 获取 web 端报告 url

```java
public Object getWebReportUrl(String taskId, Integer projectId) throws GeneralException
```

**转发目标**：`action=report, op=Report.url`（前缀 `RealWeb`）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskId | String | 是 | 任务 id |
| projectId | Integer | 是 | 项目 id |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Object | 报告 url（`result` 字段） |

**调用者**：`service/quartz/Report.java`（定时任务报告通知）

### 2. getAppReportUrl — 获取 app 端报告 url

```java
public Object getAppReportUrl(String taskId, Integer projectId) throws GeneralException
```

**转发目标**：`action=report, op=Report.url`（前缀 `RealTest`）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskId | String | 是 | 任务 id |
| projectId | Integer | 是 | 项目 id |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Object | 报告 url（`result` 字段） |

**调用者**：`service/quartz/Report.java`

### 3. getTaskIdByShareId — 根据分享 skey 反查任务 id

```java
public String getTaskIdByShareId(String shareId) throws GeneralException
```

**用途**：分享链接打开报告时，用 skey 换真实 taskid；jsonObjInfo 无 taskid 返回 null。

**转发目标**：`action=app, op=Task.shareInfo`（前缀 `RealTest`）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| shareId | String | 是 | 分享 skey |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| taskid | String | 真实任务 id（objInfo.taskid；无 taskid 返回 null） |

**调用者**：`service/task/Task.java` 等 5 处、`service/report/Report.java` 等 12 处、`service/report/Excel.java`

### 4. getShareKey — 获取分享的 skey

```java
public String getShareKey(String taskId, Integer projectId, Integer type, Integer userId, Integer eid)
```

**用途**：为任务报告生成分享 skey（type 取 ShareConfigEnum，如 WEB_PC_REPORT_SHARE），通知邮件/消息中带分享链接。

**转发目标**：`action=app, op=Task.share`（前缀 `RealTest`）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskId | String | 是 | 任务 id |
| projectId | Integer | 是 | 项目 id |
| type | Integer | 是 | 分享类型（ShareConfigEnum，如 WEB_PC_REPORT_SHARE） |
| userId | Integer | 是 | 用户 id |
| eid | Integer | 是 | 企业 id |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | String | 分享 skey |

**调用者**：`NoticeServiceImpl.java` /  /  / 

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealTest](../../../app处理服务（real-test）/07-开放接口文档/00-模块索引.md) / [RealWeb 服务](../../00-首页.md)
- [service-McPcTaskApi](service-McPcTaskApi.md)
