# service-ScriptApi — 最新脚本查询代理（api.script 版）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/script/ScriptApi.java`（@Component，继承 `AbstractApi`）
> 类型：远端代理（→ Script 服务）
> 转发方式：V1 ApiServlet 前缀 `Script`，`action=script`
> 注意：与 `api/file/script/ScriptApi.java`（[service-ScriptApi-file](service-ScriptApi-file.md)）同名不同包，本类返回 `NewScriptFile`

## 方法列表

### 1. getLasterScriptByScriptNo — 按 scriptNo 取最新脚本

```java
public NewScriptFile getLasterScriptByScriptNo(Integer projectId, Integer scriptNo)
public NewScriptFile getLasterScriptByScriptNo(Integer projectId, Integer scriptNo, Integer ignoreCheck)
```

**用途**：查询项目组下某脚本编码的最新版本脚本（两参为重载，ignoreCheck 传 null）。`ignoreCheck` 用于跳过脚本校验（定时任务重测场景传 `CommonConstant.IGNORE_SCRIPT_CHECK`）。

**转发目标**：

```java
reqJson.put("action", "script");
reqJson.put("op", "Script.getLastestScriptByScriptNo");
// data: projectid / scriptNo / ignoreCheck
```

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectId | Integer | 是 | 项目 id，null 抛 `paraInvalid` |
| scriptNo | Integer | 是 | 脚本号，null/<=0 抛 `paraInvalid` |
| ignoreCheck | Integer | 否 | 是否跳过脚本校验（两参重载传 null） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | NewScriptFile | 最新脚本对象（字段见 Script 服务，代码未确认） |

**调用者**：
- `WebQuartz.java / 742` — Web 定时任务执行/重测取最新脚本
- `McPcQuartz.java / 793` — PC 定时任务同场景

## 相关文档

- [00-分支索引](00-分支索引.md)
- [Script](../../../脚本服务/00-首页.md)
- [service-ScriptApi-file](service-ScriptApi-file.md)（同名类的更多查询方法）
- [WebQuartz](WebQuartz.md) / [McPcQuartz](McPcQuartz.md)
