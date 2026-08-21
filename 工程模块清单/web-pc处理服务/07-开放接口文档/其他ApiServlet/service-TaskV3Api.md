# service-TaskV3Api — 门户任务分页查询（V3）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/task/TaskV3Api.java`（@Component）
> 类型：远端代理（→ RealLogfile 服务）
> 转发方式：V3 REST，经 `ServiceRemoteV3Api.remoteGet`，域名 `Config.LOG_FILE_URL`（= RealLogfile 服务地址）

## 方法列表

### 1. taskList — 查询门户任务列表

```java
public PageResponseDTO<PortalTask> taskList(TaskQueryRequestDTO requestDTO) throws GeneralException
```

**用途**：按 jobId + projectId + eid + bizCode 分页查询任务关联的门户任务（定时任务重测/查询场景使用）。

**转发目标**：

```java
String url = serviceRemoteV3Api.getUrl(Config.LOG_FILE_URL + ApiUrl.LIST_TASK_URL, param);
// GET {RealLogfile}/v3/task/tasks/list?job_id=&project_id=&eid=&page=&page_size=&biz_code=
```

**请求参数**（`TaskQueryRequestDTO`，query string）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| jobId | Integer | 是 | 定时任务 id（`.toString()` 无 null 校验） |
| projectId | Integer | 是 | 项目 id |
| eid | Integer | 是 | 企业 id |
| bizCode | Integer | 是 | 业务编码 |
| page | Integer | 否 | 页码，缺省 1 |
| pageSize | Integer | 否 | 每页大小，缺省默认页大小 |

**返回参数**：`PageResponseDTO<PortalTask>`；为 null 时抛 `apiInvalid("获取任务关联信息失败")`。

| 字段 | 类型 | 说明 |
|------|------|------|
| page | Integer | 当前页 |
| pageSize | Integer | 每页大小 |
| totalPage | Integer | 总页数 |
| totalRow | Integer | 总行数 |
| list | List&lt;PortalTask&gt; | 门户任务列表（字段见 RealLogfile 服务，代码未确认） |

**调用者**：
- `BaseQuartz.java` — 定时任务公共重测逻辑
- `WebQuartz.java` / `McPcQuartz.java` — Web/PC 端重测查询任务列表
- `AppQuartz.java` — App 端重测

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealLogfile](../../../平台基础功能服务/00-首页.md)
- [WebQuartz](WebQuartz.md) / [McPcQuartz](McPcQuartz.md)
- [service-PortalTaskApi](service-PortalTaskApi.md)（V1 对等接口）
