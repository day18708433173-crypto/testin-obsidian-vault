# service-LogFile — 真机日志文件查询接口

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/real/LogFile.java`
> 类：`cn.testin.service.real.LogFile extends GenericBaseService`（非 Spring MVC Controller，无注解路由）
> 入口方式：ApiServlet 入口，`action=real`，`op=LogFile.方法名` 反射调用
> - **action**: `real`（对应包 `cn.testin.service.real`）
> - **入口格式**：`{"op": "LogFile.方法名", "action": "real", "data": {...}}`
> 依赖：`ILogService`（Spring Bean，继承自 `GenericBaseService`，真机日志文件底层服务）
> 业务：真机测试场景下的远程日志文件内容查询，支持全文匹配、分页列表、行范围获取、上下文获取四种模式。
> 涉及表：无（直接操作远程日志文件，非数据库查询）

## 方法列表总表

| # | 方法 | 说明 | 主要依赖 |
|---|---|---|---|
| 1 | match | 日志文件内容匹配（支持正则、二次匹配） | ilogservice.match |
| 2 | list | 日志文件分页查询（支持正则过滤、上下翻页） | ilogservice.list |
| 3 | range | 取指定行号范围的日志代码块 | ilogservice.range |
| 4 | context | 取当前行上下文的日志内容 | ilogservice.context |

统一返回：JSON 字符串，结构 `{ code, msg, data }`；`data` 内含 `result`（match）或 `list`（list/range/context）。
参数校验失败以 `throw new GeneralException` 抛出异常。
`logUrl` 必须包含 `Config.COOKIE_DOMAIN`（域名白名单校验）。

---

## 分发机制

- 入口：`/*`（ApiServlet）
- `action` 参数 = `real`（定位到 `cn.testin.service.real` 子包）
- `op` 参数 = `LogFile.<方法名>`（反射调用对应 public 方法）
- 请求体中 `reqjson` 为业务 JSON

---

## 1. op=LogFile.match — 日志文件内容匹配

### 请求格式
{"op": "LogFile.match", "action": "real", "data": {"logUrl": "...", "qkey": "...", "regexKeys": [...]}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| logUrl | 是 | 日志文件地址 URL，必须包含 `Config.COOKIE_DOMAIN` 白名单校验 |
| qkey | 否 | 日志匹配关键字，用于二次匹配 |
| regexKeys | 否 | 正则匹配表达式数组（JSONArray） |

### 响应结构

`data.result` = 匹配结果（String）。

### 实现意图

对指定日志文件执行内容匹配，支持传入一组正则表达式（regexKeys）做正则匹配，同时支持 qkey 做二次过滤。用于在真机测试日志中快速定位特定错误或关键字。

```java
String result = this.ilogservice.match(logUrl, qkey, regexKeys);
```

---

## 2. op=LogFile.list — 日志文件分页列表查询

### 请求格式
{"op": "LogFile.list", "action": "real", "data": {"logUrl": "...", "qkey": "...", "pageSize": ..., "startNum": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| logUrl | 是 | 日志文件地址 URL |
| qkey | 是 | 日志匹配关键字（空抛 GeneralException） |
| regexExpression | 否 | 正则表达式过滤 |
| startNum | 否 | 开始日志行号（向上翻页时必填） |
| pageSize | 否 | 每页大小，默认 200；范围 (1, 2000] |
| previous | 否 | >0 表示向上翻页（此时 startNum 必填） |

### 响应结构

`data.list` = `List<LogInfo>`（空时返回空 JSONArray）。

### 实现意图

按关键字（qkey）在远程日志文件中进行分页检索。支持正则表达式预过滤（regexExpression）、向前/向后翻页。

- `previous > 0`：向上翻页模式，需提供 `startNum`（当前行号），`Page.turnOpr = false`
- 否则向下翻页（默认），`Page.turnOpr = true`

```java
Page page = new Page();
page.setPageSize(pageSize);
page.setStartNum(startNum);
if (previous > 0) {
    page.setTurnOpr(false);   // 向上翻页
}
List<LogInfo> logs = this.ilogservice.list(logUrl, qkey, regexExpression, page);
```

---

## 3. op=LogFile.range — 取指定行范围日志

### 请求格式
{"op": "LogFile.range", "action": "real", "data": {"logUrl": "...", "start": ..., "end": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| logUrl | 是 | 日志文件地址 URL |
| regexExpression | 否 | 正则表达式过滤 |
| start | 是 | 开始行号，正整数 |
| end | 是 | 结束行号，正整数 |

### 响应结构

`data.list` = `List<LogInfo>`（空时返回空 JSONArray）。

### 实现意图

获取日志文件中指定行号范围内的内容，支持正则表达式预过滤。类似 `sed -n 'start,endp'` 的行为。

```java
List<LogInfo> list = this.ilogservice.range(logUrl, regexExpression, start, end);
```

---

## 4. op=LogFile.context — 取日志上下文

### 请求格式
{"op": "LogFile.context", "action": "real", "data": {"logUrl": "...", "curNum": ..., "pageSize": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| logUrl | 是 | 日志文件地址 URL |
| regexExpression | 否 | 正则表达式过滤 |
| curNum | 是 | 当前行号，必须 >= 1 |
| pageSize | 否 | 上下文行数（上下各取 pageSize 行），默认 200 |

### 响应结构

`data.list` = `List<LogInfo>`（空时返回空 JSONArray）。

### 实现意图

以 `curNum` 指定行为中心，获取该行上下各 `pageSize` 行的日志内容（共约 2 * pageSize 行）。支持正则表达式预过滤。典型场景：点击某行日志后查看其前后上下文。

```java
List<LogInfo> list = this.ilogservice.context(logUrl, regexExpression, curNum, pageSize);
```

---

## 返回参数（LogInfo 元素结构）

`list`/`result` 中每条日志元素 `LogInfo`（`cn.testin.pojo.log.LogInfo`）字段如下：

| 字段 | 类型 | 说明 |
|---|---|---|
| curNum | Integer | 当前日志行号 |
| info | String | 该行日志内容 |
| regexInfos | Array\<String\> | 正则匹配到的片段数组（未命中时为 null） |

## 备注

- 四个方法均对 `logUrl` 做域名白名单校验（`logUrl.contains(Config.COOKIE_DOMAIN)`），防止 SSRF 攻击。
- `list` 和 `context` 对 `pageSize` 做了范围校验（1 < pageSize < 2000），非法值用 GeneralException 抛出。
- `LogInfo` 是 pojo 对象（`cn.testin.pojo.log.LogInfo`），包含日志行号、内容等字段，底层由 `ILogService` 从远程真机返回的日志文件中解析。
- 此类不涉及数据库操作，所有数据均来自远程日志文件读取。

相关文档：[00-分支索引](00-分支索引.md)
