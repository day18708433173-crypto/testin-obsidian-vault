# service-PortalApi — 门户任务上报与查询

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/portal/PortalApi.java`（SpringHelper Bean `portal.PortalApi`）
> 类型：远端代理（→ RealPortal 服务）
> 转发方式：V1 ApiServlet 前缀（`ApiUtil.doPress("RealPortal", reqJson)`，action/op 反射路由）

## 方法列表

### 1. report — 上报任务信息到 portal

```java
public Integer report(PortalTask portalTask) throws GeneralException
```

**用途**：任务创建/状态变化时，把 PortalTask（进度、结果等）上报到真机门户，供门户页展示任务进度。

**转发目标**：`action=portal, op=Task.report`（RealPortal）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| portalTask | PortalTask | 是 | 门户任务对象（gson 序列化为 data），null 返回 0 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Integer | 上报结果 |

**调用者**：
- `TaskServiceImpl.java` /  / 
- `NoticeServiceImpl.java`

### 2. getPortalTask — 查询单个门户任务

```java
public PortalTask getPortalTask(PortalTask portalTask) throws GeneralException
```

**用途**：按条件查询门户任务详情（含 scriptTotalExecTime 等扩展信息）；jsonObjInfo 为空时返回空 PortalTask 并记 errorLog。

**转发目标**：`action=portal, op=Task.get`（RealPortal）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| portalTask | PortalTask | 是 | 门户任务查询条件对象（gson 序列化为 data），null 抛 GeneralException |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | PortalTask | 门户任务对象（含 scriptTotalExecTime 等，字段见 RealPortal 服务，代码未确认） |

**调用者**：
- `RealWebApi.java`（Web 端任务详情）
- `McPcTaskApi.java`（PC 端任务详情）
- `NoticeServiceImpl.java`

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealPortal 服务](../../../平台基础功能服务/00-首页.md)
- [service-McPcTaskApi](service-McPcTaskApi.md)、[service-RealWebApi](service-RealWebApi.md)
