# service-TaskApi — 任务调度初始化/取消/查询代理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/scheduling/TaskApi.java`（继承 `AbstractApi`，Spring bean 名 `scheduling.TaskApi`）
> 类型：远端代理（→ RealScheduling 服务）
> 转发方式：V1 ApiServlet 前缀 `RealScheduling`，统一 `action=scheduling`，op 为 `Task.*`

## 方法列表

### 1. init — 初始化任务

```java
public Integer init(JSONObject contentJson) throws GeneralException
```

**用途**：任务创建后将任务内容下发给 RealScheduling 做设备匹配与分发初始化。

**转发目标**：`op=Task.init`，contentJson 整体作为 data 透传。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| contentJson | JSONObject | 是 | 任务内容（整体作为 data 透传），null 返回 null |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Integer | 初始化结果（`result`，可为 null） |

**调用者**：
- `TaskServiceImpl.java` — 普通任务创建后初始化
- `NoticeServiceImpl.java / 5783` — 通知触发型任务初始化

### 2. cancel — 取消任务

```java
public Integer cancel(String taskid, String subtaskid, String crossTaskid, JSONObject taskGroup)
```

**转发目标**：`op=Task.cancel`，data 含 `taskid`，可选 `subtaskid/crossTaskid/taskGroup`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是 | 任务 id，空返回 null |
| subtaskid | String | 否 | 子任务 id |
| crossTaskid | String | 否 | 跨端任务 id |
| taskGroup | JSONObject | 否 | 任务组信息 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Integer | 取消结果（`result`，可为 null） |

**调用者**：
- `TaskServiceImpl.java` — 取消子任务（带 taskGroup）
- `NoticeServiceImpl.java` — 取消跨端任务（带 crossTaskid）

### 3. list — 查询任务是否存在

```java
public Integer list(String taskid) throws GeneralException
```

**转发目标**：`op=Task.list`，data 固定 `taskid + page=1 + pageSize=10`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是 | 任务 id，空返回 null |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Integer | 结果列表长度（仅用于判断任务是否已存在） |

**调用者**：`NoticeServiceImpl.java` — init 前判重。

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealScheduling](../../../任务调度服务（real-scheduling）/07-开放接口文档/00-模块索引.md)
- [service-McPcTaskApi](service-McPcTaskApi.md)（本地创建任务后走 init）
