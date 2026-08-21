# service-ScriptApi-file — 脚本文件查询代理（file.script 版）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/file/script/ScriptApi.java`（继承 `AbstractApi`）
> 类型：远端代理（→ Script 服务）
> 转发方式：V1 ApiServlet 前缀 `Script`，统一 `action=script`
> 注意：与 `api/script/ScriptApi.java`（[service-ScriptApi](service-ScriptApi.md)）同名不同包，本类返回 `ScriptFile`，方法更多

## 方法列表

### 1. listLastestScriptByScriptNo — 批量取最新版本脚本

```java
public List<ScriptFile> listLastestScriptByScriptNo(Integer projectid, List<Integer> scriptNoList)
```

**转发目标**：`op=Script.listLastestScriptByScriptNo`，data 含 `projectid/scriptNoArray`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目 id，null 抛 `paraInvalid` |
| scriptNoList | List&lt;Integer&gt; | 否 | 脚本 scriptNo 列表，空返回空列表 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | ScriptFile | 脚本文件对象（字段见 Script 服务，代码未确认） |

**调用者**：`AbstractJobUpdateStrategy.java / 143`、`ScriptSummaryServiceImpl.java / 1559`。

### 2. getLastestScriptByScriptNo — 单个取最新版本脚本

```java
public ScriptFile getLastestScriptByScriptNo(Integer projectid, Integer scriptNo)
```

**转发目标**：`op=Script.getLastestScriptByScriptNo`，data 含 `projectid/scriptNo`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目 id，null 抛 `paraInvalid` |
| scriptNo | Integer | 是 | 脚本号，null/<=0 抛 `paraInvalid` |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | ScriptFile | 脚本文件对象（字段见 Script 服务，代码未确认） |

### 3. listByScriptids — 按脚本 id 批量查询

```java
public List<ScriptFile> listByScriptids(Integer projectid, List<Integer> scriptidList)
```

**转发目标**：`op=Script.listByScriptids`，data 含 `projectid/scriptIdArray`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目 id，null 抛 `paraInvalid` |
| scriptidList | List&lt;Integer&gt; | 否 | 脚本 id 列表，空返回空列表 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | ScriptFile | 脚本文件对象（字段见 Script 服务，代码未确认） |

**调用者**：`TaskServiceImpl.java`。

### 4. findScriptByScriptIdList — 按脚本 id 列表查询（不限项目）

```java
public List<ScriptFile> findScriptByScriptIdList(List<Integer> scriptids)
```

**转发目标**：`op=Script.findScriptByScriptIdList`，data 含 `scriptids` 数组。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| scriptids | List&lt;Integer&gt; | 是 | 脚本 id 列表，空抛 `paraInvalid` |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | ScriptFile | 脚本文件对象（字段见 Script 服务，代码未确认） |

**调用者**：`ScriptSummaryServiceImpl.java`。

### 5. findFinalScriptByScriptNoList — 按脚本编码查最新有效脚本

```java
public List<ScriptFile> findFinalScriptByScriptNoList(List<Integer> scriptNos)
```

**转发目标**：`op=Script.findFinalScriptByScriptNoList`，data 含 `scriptnos`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| scriptNos | List&lt;Integer&gt; | 是 | 脚本编码列表，空抛 `paraInvalid` |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | ScriptFile | 脚本文件对象（字段见 Script 服务，代码未确认） |

**调用者**：`NoticeServiceImpl.java / 1021 / 1066`、`ScriptSummaryServiceImpl.java / 389`。

### 6. findScriptStepByScriptId — 查询脚本步骤

```java
public List<ScriptStep> findScriptStepByScriptId(Integer scriptid)
```

**转发目标**：`op=ScriptStep.list`，data 含 `scriptid`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| scriptid | Integer | 是 | 脚本 id，null/<=0 抛 `paraInvalid` |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | ScriptStep | 脚本步骤对象（字段见 Script 服务，代码未确认） |

**调用者**：`ScriptSummaryServiceImpl.java` — 报告摘要中展开脚本步骤。

## 相关文档

- [00-分支索引](00-分支索引.md)
- [Script](../../../脚本服务/00-首页.md)
- [service-ScriptApi](service-ScriptApi.md)（api.script 同名类）
