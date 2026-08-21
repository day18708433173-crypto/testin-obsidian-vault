# service-ScriptGroupV3Api — 脚本组展开 scriptNo 代理（V3）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/script/ScriptGroupV3Api.java`（@Service）
> 类型：远端代理（→ Script 服务）
> 转发方式：V3 REST，经 `ServiceRemoteV3Api.remotePost`；域名 `Config.FILE_SYSTEM_URL`（= Script 服务地址）

## 方法列表

### 1. getScriptNoByGroupIds — 按组 id 列表取全部 scriptNo

```java
public List<Integer> getScriptNoByGroupIds(List<Integer> groupId) throws GeneralException
```

**用途**：把脚本组 id 列表展开为组内全部脚本编码，用于任务创建时 scripts 参数归集；groupId 为空时返回空列表。

**转发目标**：

```java
String url = Config.FILE_SYSTEM_URL + ApiUrl.SCRIPT_GROUT_NUM_URL;
// POST {Script}/v3/script/script_groups/script_no
// body: ScriptGroupQueryRequest(groupId)
```

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| groupId | List&lt;Integer&gt; | 否 | 脚本组 id 列表，空返回空列表 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | Integer | 组内全部脚本 scriptNo 列表 |

**调用者**：
- `TaskServiceImpl.java` — 任务创建时展开脚本组
- `TemplateService.java` — 模板场景展开脚本组

## 相关文档

- [00-分支索引](00-分支索引.md)
- [Script](../../../脚本服务/00-首页.md)
- [service-ScriptGroupApi](service-ScriptGroupApi.md)（V1 版）
