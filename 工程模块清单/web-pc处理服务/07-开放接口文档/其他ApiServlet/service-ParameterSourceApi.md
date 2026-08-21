# service-ParameterSourceApi — 参数数据源代理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/file/script/ParameterSourceApi.java`（继承 `AbstractApi`，多为 static 方法）
> 类型：远端代理（→ DataSource 为主，部分 → Script / RealTest 服务）
> 转发方式：V1 ApiServlet（前缀字面量 `DataSource` / `Script` / `RealTest`）+ 一处 HttpClient 直连 V3

## 转发目标总览

| 方法 | 前缀 | action / op |
|------|------|-------------|
| listByScriptNo | DataSource | source / DataTableCtrl.getScriptParamData |
| listScriptParamDataByScriptNoNew | DataSource | source / DataTableCtrl.getScriptParamData |
| listScriptParamSourceNew | DataSource | source / SourceConfigCtrl.selectSourceConfig |
| getTagInfoList | DataSource | source / TagInfoCtrl.selectAll |
| getParamTableInfo | DataSource（HttpClient） | POST /v3/datasource/executive_summaries |
| getScriptParamDataSource | Script | script / ParamDataSourceService.getScriptParamDataSource |
| listScriptParamData | Script | script / ParamDataSourceService.listScriptParamData |
| listSourceByParentId | Script | script / ParamDataSourceService.getSourceByParentId |
| listScriptParamDataByScriptNo | Script | script / ParamDataSourceService.listScriptParamDataByScriptNo |
| getDefaultDataDistribute | RealTest | app / ParamSource.assign |

## 方法列表

### 1. listByScriptNo — 按 scriptNo 列表查脚本参数数据

```java
public List<ScriptParamDataInfo> listByScriptNo(Integer projectid, String sourceId, List<Integer> scriptNoList)
```

**转发目标**：DataSource `DataTableCtrl.getScriptParamData`，data 含 `projectid/sourceConfigId/scriptNoList`。返回后把 `id` 回填为 `scriptNo`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目 id，null/<=0 返回 null |
| sourceId | String | 是 | 数据源 id，空返回 null |
| scriptNoList | List&lt;Integer&gt; | 是 | 脚本 scriptNo 列表，空返回 null |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | ScriptParamDataInfo | 脚本参数数据对象（字段见 DataSource 服务，代码未确认） |

### 2. listScriptParamDataByScriptNoNew — 带标签/行过滤的参数数据查询

```java
public static List<ScriptParamDataInfo> listScriptParamDataByScriptNoNew(
    Integer projectid, Integer sourceId, List<Integer> scriptNoList, Integer id,
    List<Integer> tagList, List<Integer> skipTagList, Map<Integer, Set<Integer>> scriptNoRowIds)
```

**转发目标**：同上 op；额外透传 `tagList/noHasTagList(←skipTagList)/scriptNoRowIds`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目 id，null/<=0 返回 null |
| sourceId | Integer | 否 | 数据源 id |
| scriptNoList | List&lt;Integer&gt; | 否 | 脚本 scriptNo 列表 |
| id | Integer | 否 | 参数数据 id |
| tagList | List&lt;Integer&gt; | 否 | 标签 id 列表 |
| skipTagList | List&lt;Integer&gt; | 否 | 跳过标签列表（→ noHasTagList） |
| scriptNoRowIds | Map&lt;Integer,Set&lt;Integer&gt;&gt; | 否 | 脚本号-行号映射 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | ScriptParamDataInfo | 脚本参数数据对象（字段见 DataSource 服务，代码未确认） |

**调用者**：`IParamDataSourceServiceImpl.java` — 任务创建时解析参数数据源。

### 3. getScriptParamDataSource — 获取数据源配置 JSON

```java
public static String getScriptParamDataSource(Integer sourceId)
```

**转发目标**：Script `ParamDataSourceService.getScriptParamDataSource`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| sourceId | Integer | 是 | 数据源 id |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | String | 数据源配置 JSON 字符串（objInfo 序列化；为空时返回 `{}`） |

**调用者**：`TaskServiceImpl.java`。

### 4. listScriptParamData / listSourceByParentId / listScriptParamDataByScriptNo

均走 Script 服务 `ParamDataSourceService.*`：分别按 sourceId+projectId+appId 查参数数据列表、按 parentId 查子数据源、按 scriptNo 列表查参数数据。

**4.1 listScriptParamData(ScriptParamData condition)** — 查参数数据列表

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| condition.sourceId | String | 否 | 数据源 id |
| condition.projectId | Integer | 否 | 项目 id |
| condition.appId | Integer | 否 | 应用 id |

返回：`List<ScriptParamData>`（元素字段见 Script 服务，代码未确认）。

**4.2 listSourceByParentId(Integer projectId, Integer eid, Integer parentId)** — 按父数据源查子数据源

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectId | Integer | 否 | 项目 id |
| eid | Integer | 否 | 企业 id |
| parentId | Integer | 否 | 父数据源 id |

返回：`List<ScriptParamSource>`（元素字段见 Script 服务，代码未确认）。

**4.3 listScriptParamDataByScriptNo(Integer projectid, String sourceId, List<Integer> scriptNoList)** — 按 scriptNo 列表查参数数据

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目 id，null/<=0 返回 null |
| sourceId | String | 是 | 数据源 id，空返回 null |
| scriptNoList | List&lt;Integer&gt; | 是 | 脚本 scriptNo 列表，空返回 null |

返回：`List<ScriptParamData>`（元素字段见 Script 服务，代码未确认）。

### 5. listScriptParamSourceNew — 查询数据源配置列表

```java
public static List<SourceConfig> listScriptParamSourceNew(ScriptParamSource condition, List<Integer> tagList)
```

**转发目标**：DataSource `SourceConfigCtrl.selectSourceConfig`，data 含 `projectid/type/sourceId/scriptNo?/tagList?`。

**请求参数**（`ScriptParamSource condition` + `tagList`）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| condition.projectId | Integer | 否 | 项目 id |
| condition.type | Integer | 否 | 数据源类型（转 Integer 透传） |
| condition.sourceId | String | 否 | 数据源 id |
| condition.scriptNo | Integer | 否 | 脚本号 |
| tagList | List&lt;Integer&gt; | 否 | 标签 id 列表 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | SourceConfig | 数据源配置对象（字段见 DataSource 服务，代码未确认） |

**调用者**：`IParamDataSourceServiceImpl.java / 331`。

### 6. getDefaultDataDistribute — 参数默认分配

```java
public static JSONArray getDefaultDataDistribute(Integer projectid, Integer type, String paramSource,
                                                 JSONArray devices, JSONArray scripts, List<Integer> tagList)
```

**转发目标**：RealTest `ParamSource.assign`；`type`：1=局部变量分配，2=全局参数分配；devices 为空直接返回 null。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 否 | 项目 id |
| type | Integer | 否 | 类型（1=局部变量，2=全局参数） |
| paramSource | String | 否 | 参数数据源 |
| devices | JSONArray | 是 | 设备列表，空直接返回 null |
| scripts | JSONArray | 否 | 脚本列表 |
| tagList | List&lt;Integer&gt; | 否 | 标签 id 列表 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (数组) | JSONArray | 默认分配结果数组（元素结构见 RealTest 服务，代码未确认） |

**调用者**：`IParamDataSourceServiceImpl.java / 267`。

### 7. getTagInfoList — 按标签 id 查标签信息

```java
public static List<TagInfo> getTagInfoList(List<Integer> tagList)
```

**转发目标**：DataSource `TagInfoCtrl.selectAll`，data 含 `tagIdList`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| tagList | List&lt;Integer&gt; | 否 | 标签 id 列表 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | TagInfo | 标签信息对象（含 name，完整字段见 DataSource 服务，代码未确认） |

**调用者**：`RealWebApi.java / 255`、`McPcTaskApi.java / 197`、`Task.java / 928`（任务详情补充标签名）。

### 8. getParamTableInfo — 查表格列属性与行标签

```java
public List<SourceColAndTag> getParamTableInfo(Integer sourceId, List<Integer> scriptNoList, List<Integer> tagList)
```

**转发目标**：HttpClient `POST {DataSource}/v3/datasource/executive_summaries`，body 含 `scriptNos/sourceId/tagList`；DataSource 域名缓存于静态字段 `DATASOURCEURL`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| sourceId | Integer | 否 | 数据源 id |
| scriptNoList | List&lt;Integer&gt; | 否 | 脚本号列表 |
| tagList | List&lt;Integer&gt; | 否 | 标签 id 列表 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | SourceColAndTag | 表格列属性与行标签对象（字段见 DataSource 服务，代码未确认） |

## 相关文档

- [00-分支索引](00-分支索引.md)
- [DataSource](../../../数据源/00-首页.md)、[Script](../../../脚本服务/00-首页.md)、[RealTest](../../../app处理服务（real-test）/07-开放接口文档/00-模块索引.md)
- [service-McPcTaskApi](service-McPcTaskApi.md) / [service-RealWebApi](service-RealWebApi.md)
