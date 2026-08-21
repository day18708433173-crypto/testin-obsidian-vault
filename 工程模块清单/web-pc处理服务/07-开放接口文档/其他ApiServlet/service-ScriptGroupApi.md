# service-ScriptGroupApi — 脚本组查询代理（file.script 版）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/file/script/ScriptGroupApi.java`（继承 `AbstractApi`）
> 类型：远端代理（→ Script 服务）
> 转发方式：V1 ApiServlet 前缀 `Script`，统一 `action=script`，op 为 `ScriptGroup.*`

## 方法列表

### 1. list — 分页查询脚本组

```java
public BaseList<ScriptGroup> list(Map<String, Object> conditionMap, Integer page, Integer pageSize)
```

**用途**：按条件分页查询脚本组；conditionMap 支持 `projectid/appId/scriptGroupId/creator/scriptGroupDesc`，page/pageSize 会被并入 conditionMap 透传。

**转发目标**：`op=ScriptGroup.list`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| conditionMap | Map&lt;String,Object&gt; | 是 | 查询条件（projectid/appId/scriptGroupId/creator/scriptGroupDesc），null 或空抛 `paraInvalid` |
| page | Integer | 是 | 页码，null/<=0 抛 `paraInvalid` |
| pageSize | Integer | 是 | 每页大小，null/<=0 抛 `paraInvalid` |

**返回参数**：`BaseList<ScriptGroup>`，回填 curPage/pageSize/totalRow/totalPage。

| 字段 | 类型 | 说明 |
|------|------|------|
| list | List&lt;ScriptGroup&gt; | 脚本组列表 |
| curPage | Integer | 当前页 |
| pageSize | Integer | 每页大小 |
| totalRow | Integer | 总行数 |
| totalPage | Integer | 总页数 |

### 2. getScriptGroupById — 按组 id 查询脚本组

```java
public ScriptGroup getScriptGroupById(Integer projectid, Integer groupid)
```

**转发目标**：`op=ScriptGroup.get`，data 含 `projectid/groupId`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目 id，null/<=0 返回 null |
| groupid | Integer | 是 | 脚本组 id，null/<=0 返回 null |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | ScriptGroup | 脚本组对象（字段见 Script 服务，代码未确认） |

**调用者**：`AbstractJobUpdateStrategy.java` — 定时任务更新策略中按 groupId 取脚本组展开脚本。

## 相关文档

- [00-分支索引](00-分支索引.md)
- [Script](../../../脚本服务/00-首页.md)
- [service-ScriptGroupOperateApi](service-ScriptGroupOperateApi.md)（同 op 的另一封装）
- [service-ScriptGroupV3Api](service-ScriptGroupV3Api.md)（V3 版）
