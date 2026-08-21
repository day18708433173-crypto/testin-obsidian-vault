# service-ProjectApi — 项目组信息查询代理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/user/ProjectApi.java`（继承 `AbstractApi`，Spring bean 名 `user.ProjectApi`）
> 类型：远端代理（→ UserManager 服务）
> 转发方式：V1 ApiServlet 前缀 `UserManager`，`ApiUtil.doPress(userPrefixName, reqJson)`

## 方法列表

### 1. get — 按项目组 id 查询

```java
public ProjectInfo get(Integer projectid) throws GeneralException
```

**用途**：通过 projectid 获取项目组信息（project_info）。

**转发目标**：`action=user, op=Project.get`，data 仅含 `projectid`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组 id，null/<=0 返回 null |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | ProjectInfo | 项目组信息对象（字段见 UserManager 服务，代码未确认） |

**调用者**：
- `TestPlanExcelService.java` — 导出执行记录 Excel 时取项目名
- `GenerateReportServiceImpl.java` — 生成报告时取项目信息

### 2. getUserProjectList — 查询用户项目组列表

```java
public List<ProjectInfo> getUserProjectList(Integer eid, Integer userid, Integer page, Integer pageSize)
```

**用途**：按用户 id + 企业 id 分页查询其可见项目组列表。page 缺省 1，pageSize 缺省 999。

**转发目标**：`action=user, op=Project.getUserProjectList`，data 含 `userid/eid/page/pageSize`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业 id，null/<=0 返回 null |
| userid | Integer | 是 | 用户 id，null/<=0 返回 null |
| page | Integer | 否 | 页码，缺省 1 |
| pageSize | Integer | 否 | 每页大小，缺省 999 |

**返回参数**：`List<ProjectInfo>`（遍历 `jsonList` 逐条 gson 反序列化）

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | ProjectInfo | 项目组信息对象（字段见 UserManager 服务，代码未确认） |

**调用者**：
- `GenericBaseService.java` — `projectapi.getUserProjectList(userid, eid, null, null)`
- `TaskServiceImpl.java` — 任务创建前校验用户项目组

## 相关文档

- [00-分支索引](00-分支索引.md)
- [UserManager](../../../平台基础功能服务/00-首页.md)
- [service-UserApi](service-UserApi.md)
