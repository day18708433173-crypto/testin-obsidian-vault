# service-QcCfg — QC 质量中心配置接口

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/qc/QcCfg.java`
> 类：`cn.testin.service.qc.QcCfg extends GenericBaseService`（非 Spring MVC Controller，无注解路由）
> 入口方式：ApiServlet 入口，`action=qc`，`op=QcCfg.方法名` 反射调用；各方法返回 JSON 字符串（`ApiUtil.getJSONobj / getResult`）
> - **action**: `qc`（对应包 `cn.testin.service.qc`）
> - **入口格式**：`{"op": "QcCfg.方法名", "action": "qc", "data": {...}}`
> 依赖：`IQcCfgService`（Spring Bean，继承自 `GenericBaseService`）
> 业务：QC 质量中心渠道配置的 CRUD（列表查询、新增、修改、删除、单条获取）。
> 涉及表：`db_notice.qc_cfg`

## 方法列表总表

| # | 方法 | 说明 | 主要依赖 |
|---|---|---|---|
| 1 | list | 分页查询 QC 配置列表（按 eid/projectid/channelName/status 条件） | iQcCfgService.list |
| 2 | add | 新增 QC 配置 | iQcCfgService.add |
| 3 | maintain | 更新 QC 配置 | iQcCfgService.update |
| 4 | del | 按 eid + projectid 删除 QC 配置 | iQcCfgService.delete |
| 5 | get | 按 eid + projectid + status 获取单条配置 | iQcCfgService.get |

统一返回：JSON 字符串，结构 `{ code, msg, data }`；`data` 内含 `result` / `list` / `object` 等键（`ApiResponse.RES_*` 常量）。
参数校验失败统一返回 `CommonCode.paraInvalid` + 具体提示（如 `eid is invalid`）。
公共分页规则：`page < 1` 归为 1；`pageSize` 为空/非法/超过 `Config.MaxSize` 时取 `Config.MaxSize`。

---

## 分发机制

- 入口：`/*`（ApiServlet）
- `action` 参数 = `qc`（定位到 `cn.testin.service.qc` 子包）
- `op` 参数 = `QcCfg.<方法名>`（反射调用对应 public 方法）
- 请求体中 `reqjson` 为业务 JSON，每个方法从其中按需提取参数

---

## 1. op=QcCfg.list — 分页查询 QC 配置列表

### 请求格式
{"op": "QcCfg.list", "action": "qc", "data": {"eid": ..., "projectid": ..., "page": ..., "pageSize": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 否 | 企业ID，传了则校验为正整数（参数无效返回 paraInvalid） |
| projectid | 否 | 项目组ID，传了则校验为非负整数 |
| channelName | 否 | 渠道名称，模糊条件 |
| status | 否 | 状态码，传了则校验为非负整数 |
| page | 否 | 页码，默认 1 |
| pageSize | 否 | 每页大小，默认/上限 `Config.MaxSize` |

### 响应结构

`data` 由 `baseListToResData` 从 `BaseList<DbQcCfg>` 转换的分页结构（含 `list`、`page`、`pageSize`、`totalRow`、`totalPage`）。

### 实现意图

分页查询 QC 配置。特殊逻辑：当按 `projectid > 0` 条件查询结果为 0 条时，自动将 projectid 改为 0 重新查询一次（即查项目组级 + 默认级配置合并返回）。

### 涉及的数据库操作

`iQcCfgService.list(conditionMap, page, pageSize)` — 表 `db_notice.qc_cfg`，条件查询 + 分页。

---

## 2. op=QcCfg.add — 新增 QC 配置

### 请求格式
{"op": "QcCfg.add", "action": "qc", "data": {"eid": ..., "projectid": ..., "channelName": "...", "serviceUrl": "...", "config": "..."}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 是 | 企业ID，正整数 |
| projectid | 是 | 项目组ID，非负整数 |
| channelName | 是 | 渠道名称，非空 |
| serviceUrl | 是 | 服务接口地址，非空 |
| config | 是 | 相关配置，非空 |
| descr | 否 | 描述 |
| status | 否 | 状态，默认 1（有效） |
| createtime | 否 | 创建时间，默认 `System.currentTimeMillis()` |

> 参数由 `DbQcCfg.toBean(reqjson)` 转换后经 `validate(qcCfg, apirequest, "add")` 校验。

### 响应结构

`data.result` = 插入结果（Integer，>0 表示成功）。

### 涉及的数据库操作

`iQcCfgService.add(qcCfg)` — 表 `db_notice.qc_cfg`，插入新记录。

---

## 3. op=QcCfg.maintain — 更新 QC 配置

### 请求格式
{"op": "QcCfg.maintain", "action": "qc", "data": {"eid": ..., "projectid": ..., "channelName": "...", ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 是 | 企业ID，正整数 |
| projectid | 是 | 项目组ID，非负整数 |
| channelName | 否 | 渠道名称 |
| serviceUrl | 否 | 服务接口地址 |
| config | 否 | 相关配置 |
| descr | 否 | 描述 |
| status | 否 | 状态 |
| updatetime | 否 | 更新时间，默认 `System.currentTimeMillis()` |

> 参数由 `DbQcCfg.toBean(reqjson)` 转换后经 `validate(qcCfg, apirequest, "maintain")` 校验（maintain 模式不强制校验 channelName/serviceUrl/config 非空）。

### 响应结构

`data.result` = 更新结果（Integer）。

### 涉及的数据库操作

`iQcCfgService.update(qcCfg)` — 表 `db_notice.qc_cfg`，更新已有记录。

---

## 4. op=QcCfg.del — 删除 QC 配置

### 请求格式
{"op": "QcCfg.del", "action": "qc", "data": {"eid": ..., "projectid": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 是 | 企业ID，正整数（无效返回 paraInvalid） |
| projectid | 是 | 项目组ID，非负整数（无效返回 paraInvalid） |

### 响应结构

`data.result` = 删除结果（Integer）。

### 涉及的数据库操作

`iQcCfgService.delete(eid, projectid)` — 表 `db_notice.qc_cfg`，按 eid+projectid 删除。

---

## 5. op=QcCfg.get — 获取单条 QC 配置

### 请求格式
{"op": "QcCfg.get", "action": "qc", "data": {"eid": ..., "projectid": ..., "status": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 是 | 企业ID，正整数 |
| projectid | 是 | 项目组ID，非负整数 |
| status | 否 | 状态筛选，值为 0 或 1 时作为查询条件 |

### 响应结构

`data.object` = `DbQcCfg` 的 JSON 对象（查不到时为空）。

### 返回参数（DbQcCfg 元素结构）

`list` 中每条 / `object` 的字段（表 `db_notice.qc_cfg`）：

| 字段 | 类型 | 说明 |
|---|---|---|
| eid | Integer | 企业ID |
| projectid | Integer | 项目组ID |
| channelName | String | 渠道名称 |
| serviceUrl | String | 服务接口地址 |
| config | String | 相关配置（JSON 字符串） |
| descr | String | 描述 |
| status | Integer | 状态（1=有效） |
| createtime | Long | 创建时间 |
| updatetime | Long | 更新时间 |

### 涉及的数据库操作

`iQcCfgService.get(eid, projectid, status)` — 表 `db_notice.qc_cfg`，按条件获取单条。

---

## 私有方法：validate

`validate(DbQcCfg, ApiRequest, String optType)` — 校验 QC 配置参数的通用校验器：

- 校验 eid（正整数）、projectid（非负整数）
- `optType = "add"` 时额外校验：channelName、serviceUrl、config 非空；status 默认 1；createtime 默认当前时间
- 任意场景：updatetime 为空时设为当前时间
- 校验失败返回 `ApiUtil.getResult` 拼装的错误 JSON；成功返回 "success" 字符串

---

## 备注

- 该类所有方法均为老门户 JSON 派发风格（ApiServlet 反射调用），参数从 `reqjson` 手工取并逐个校验，不走 `@Valid` 注解。
- `list` 方法中 projectid=0 的兜底重查逻辑：当前端传了具体项目组但有条件时，若结果为空则自动回退查项目组级（0）配置，用于配置继承场景。
- `add` 时 status 默认为 1（有效），`updatetime` 在 add 和 maintain 时均自动设为当前时间（若前端未传）。

相关文档：[00-分支索引](00-分支索引.md) · [service-QcTask](service-QcTask.md)
