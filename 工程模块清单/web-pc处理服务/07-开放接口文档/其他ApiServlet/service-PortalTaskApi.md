# service-PortalTaskApi — 门户任务列表与批量删除代理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/task/PortalTaskApi.java`（@Component，继承 `AbstractApi`）
> 类型：远端代理（→ RealPortal 服务）
> 转发方式：V1 ApiServlet 前缀 `RealPortal`，统一 `action=portal`，op 为 `Task.*`

## 方法列表

### 1. list — 按条件查询门户任务列表

```java
public PageUtils list(Integer eid, Integer projectId, JSONObject conditionJson) throws GeneralException
```

**用途**：按条件分页查询门户任务（portal_task_XX 分表）。

**转发目标**：`op=Task.list`；conditionJson 中强制写入 `eid/projectid` 后作为 data 整体透传。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业 id，null/<=0 抛 `paraInvalid` |
| projectId | Integer | 是 | 项目 id，null/<1 抛 `paraInvalid` |
| conditionJson | JSONObject | 否 | 过滤条件（null 视为空对象） |

**返回参数**：`PageUtils`

| 字段 | 类型 | 说明 |
|------|------|------|
| list | List&lt;PortalTask&gt; | 门户任务列表 |
| currPage | Integer | 当前页（取自响应 page） |
| pageSize | Integer | 每页大小 |
| totalPage | Integer | 总页数 |
| totalCount | Integer | 总行数（取自响应 totalRow） |

**调用者**：

**调用者**：
- `WebQuartz.java` / `McPcQuartz.java` — Web/PC 端任务列表查询
- `AppQuartz.java` — App 端任务列表查询

### 2. batchDelete — 批量删除门户任务

```java
public boolean batchDelete(Integer eid, Integer projectId, List<String> taskIds) throws GeneralException
```

**转发目标**：`op=Task.batchDelete`，data 含 `eid/projectid/taskids`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业 id，null/<=0 抛 `paraInvalid` |
| projectId | Integer | 是 | 项目 id，null/<1 抛 `paraInvalid` |
| taskIds | List&lt;String&gt; | 否 | 任务 id 列表，空直接返回 true 不发请求 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Boolean | 远端 result > 0 为 true，否则 false |

**调用者**：
- `WebQuartz.java` / `McPcQuartz.java` — 定时任务清理门户任务
- `BaseQuartz.java` — 公共清理逻辑

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealPortal](../../../平台基础功能服务/00-首页.md)
- [WebQuartz](WebQuartz.md) / [McPcQuartz](McPcQuartz.md)
- [service-TaskV3Api](service-TaskV3Api.md)（V3 对等接口）
