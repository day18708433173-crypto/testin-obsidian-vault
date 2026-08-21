# service-SmsCfg — 短信配置

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/sms/SmsCfg.java`
> 类：`cn.testin.service.sms.SmsCfg extends GenericBaseService`（非 Spring MVC Controller）
> 入口方式：ApiServlet 网关按 `action` 定位（`sms.SmsCfg`），`op` 反射调用方法名；`ApiRequest.reqjson` 传参；返回 JSON 字符串。
> - **action**: `sms`（对应包 `cn.testin.service.sms`）
> - **入口格式**：`{"op": "SmsCfg.方法名", "action": "sms", "data": {...}}`
> 依赖：`ISmsCfgService`（继承自 `GenericBaseService`）
> 业务：短信渠道配置（服务地址、接口参数等）的增删改查。按企业维度（eid）管理，一个 eid 可有多条短信配置（不同供应商/优先级）。

## 方法列表总表

| # | op | 方法 | 说明 |
|---|---|---|---|
| 1 | SmsCfg.list | list | 短信配置分页列表（可按 eid/channelName/status 筛选） |
| 2 | SmsCfg.add | add | 新增短信配置 |
| 3 | SmsCfg.maintain | maintain | 更新短信配置 |
| 4 | SmsCfg.del | del | 按 eid 删除短信配置 |
| 5 | SmsCfg.get | get | 按 eid（可加 status）获取单条短信配置 |

统一返回：JSON 字符串。公共分页：`page<1` 归 1，`pageSize` 超限取 `Config.MaxSize`。

---

## 1. op=SmsCfg.list — 短信配置分页列表

### 请求格式
{"op": "SmsCfg.list", "action": "sms", "data": {"eid": ..., "channelName": "...", "status": ..., "page": ..., "pageSize": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 否 | 企业 ID，传了校验 > 0 且为数字 |
| channelName | 否 | 渠道名称，模糊匹配 |
| status | 否 | 状态，传了校验 >= 0 |
| page / pageSize | 否 | 分页 |

### 响应结构

`data` 由 `baseListToResData` 从 `BaseList<DbSmsCfg>` 转换的分页结构。

### 调用链

```
SmsCfg.list → ISmsCfgService.list(conditionMap, page, pageSize)
```

---

## 2. op=SmsCfg.add — 新增短信配置

### 请求格式
{"op": "SmsCfg.add", "action": "sms", "data": {"eid": ..., "channelName": "...", "serviceUrl": "...", "config": "..."}}

### 请求参数（reqjson）

`DbSmsCfg.toBean(reqjson)` 反序列化，再由 `validate(smsCfg, "add")` 校验。

**校验规则（add 模式）：**

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 是 | 企业 ID，> 0 |
| channelName | 是 | 渠道名称，不能为空 |
| serviceUrl | 是 | 服务接口地址，不能为空 |
| config | 是 | 相关配置（JSON 字符串），不能为空 |
| status | 否 | 默认 1（启用） |
| createtime | 否 | 默认当前时间 |

### 响应结构

`data.result` = insert 条数。

### 调用链

```
SmsCfg.add → validate(smsCfg, "add") → ISmsCfgService.add(smsCfg)
```

---

## 3. op=SmsCfg.maintain — 更新短信配置

### 请求格式
{"op": "SmsCfg.maintain", "action": "sms", "data": {"id": ..., "eid": ..., ...}}

### 请求参数（reqjson）

`DbSmsCfg.toBean(reqjson)` 反序列化，再由 `validate(smsCfg, "maintain")` 校验。

**校验规则（maintain 模式）：**

与 add 的区别在于 maintain 不强制校验 channelName/serviceUrl/config 为非空（允许部分更新），但 updatetime 自动设为当前时间。

### 响应结构

`data.result` = update 影响行数。

### 调用链

```
SmsCfg.maintain → validate(smsCfg, "maintain") → ISmsCfgService.update(smsCfg)
```

---

## 4. op=SmsCfg.del — 删除短信配置

### 请求格式
{"op": "SmsCfg.del", "action": "sms", "data": {"eid": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 是 | 企业 ID，必须 > 0 且为数字 |

注意：与常见的按 id 删除不同，此处按 eid 删除（会删除该企业下所有短信配置）。

### 响应结构

`data.result` = 删除影响行数。

### 调用链

```
SmsCfg.del → ISmsCfgService.delete(eid)
```

---

## 5. op=SmsCfg.get — 获取单条短信配置

### 请求格式
{"op": "SmsCfg.get", "action": "sms", "data": {"eid": ..., "status": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 是 | 企业 ID，必须 > 0 且为数字 |
| status | 否 | 状态筛选，0-1 之间才参与查询 |

### 响应结构

- 命中：`data.object` = `DbSmsCfg` JSON。
- 未命中：`data.object` 不存在（空 map）。

### 调用链

```
SmsCfg.get → ISmsCfgService.get(eid, status)
```

---

## 备注

- 校验方法 `validate` 按 optType 区分 add/maintain：add 模式强制校验 channelName/serviceUrl/config 非空并设默认 createtime；maintain 仅校验 eid 和 updatetime。
- `del` 按 eid 整企业删除，注意误操作风险。

---

相关文档：[00-分支索引](00-分支索引.md) · [service-SmsTask](service-SmsTask.md) · [service-SmsTemplet](service-SmsTemplet.md)
