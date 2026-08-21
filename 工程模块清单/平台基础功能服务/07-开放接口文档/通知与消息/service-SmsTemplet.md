# service-SmsTemplet — 短信模板配置

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/sms/SmsTemplet.java`
> 类：`cn.testin.service.sms.SmsTemplet extends GenericBaseService`（非 Spring MVC Controller）
> 入口方式：ApiServlet 网关按 `action` 定位（`sms.SmsTemplet`），`op` 反射调用方法名；`ApiRequest.reqjson` 传参；返回 JSON 字符串。
> - **action**: `sms`（对应包 `cn.testin.service.sms`）
> - **入口格式**：`{"op": "SmsTemplet.方法名", "action": "sms", "data": {...}}`
> 依赖：`ISmsTempletService`（继承自 `GenericBaseService`）
> 业务：短信模板（模板类别、名称、内容）的增删改查。按企业维度（eid）管理，模板 key 标识业务场景类别。

## 方法列表总表

| # | op | 方法 | 说明 |
|---|---|---|---|
| 1 | SmsTemplet.list | list | 短信模板分页列表（可按 eid/key/name/descr/status 筛选） |
| 2 | SmsTemplet.add | add | 新增短信模板 |
| 3 | SmsTemplet.maintain | maintain | 更新短信模板 |
| 4 | SmsTemplet.del | del | 按 id 删除短信模板 |
| 5 | SmsTemplet.get | get | 按 id 获取短信模板 |

统一返回：JSON 字符串。公共分页：`page<1` 归 1，`pageSize` 超限取 `Config.MaxSize`。

---

## 1. op=SmsTemplet.list — 短信模板分页列表

### 请求格式
{"op": "SmsTemplet.list", "action": "sms", "data": {"eid": ..., "key": "...", "page": ..., "pageSize": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 否 | 企业 ID，> 0 才参与筛选 |
| key | 否 | 模板类别（如 `REGISTER`、`RESET_PWD` 等） |
| name | 否 | 模板名称，模糊匹配 |
| descr | 否 | 模板描述，模糊匹配 |
| status | 否 | 状态，>= 0 才参与筛选 |
| page / pageSize | 否 | 分页 |

### 响应结构

`data` 由 `baseListToResData` 转换的分页结构。

### 调用链

```
SmsTemplet.list → ISmsTempletService.list(conditionMap, page, pageSize)
```

---

## 2. op=SmsTemplet.add — 新增短信模板

### 请求格式
{"op": "SmsTemplet.add", "action": "sms", "data": {"eid": ..., "key": "...", "name": "...", "content": "..."}}

### 请求参数（reqjson）

`DbSmsTemplet.toBean(reqjson)` 反序列化，再由 `validate(smsTemplet, "add")` 校验。

**add 模式校验规则：**

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 是 | 企业 ID，> 0 |
| key | 是 | 模板类别，不能为空 |
| name | 是 | 模板名称，不能为空 |
| content | 是 | 模板内容，不能为空 |
| status | 否 | 默认 1（启用） |
| createtime | 否 | 默认当前时间 |

### 响应结构

`data.result` = insert 条数。

### 调用链

```
SmsTemplet.add → validate(smsTemplet, "add") → ISmsTempletService.add(smsTemplet)
```

---

## 3. op=SmsTemplet.maintain — 更新短信模板

### 请求格式
{"op": "SmsTemplet.maintain", "action": "sms", "data": {"id": ..., "eid": ..., ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| id | 是 | 模板主键，>= 1（在 validate 之前手动从 reqjson 提取并校验） |
| （其余字段由 DbSmsTemplet.toBean 反序列化） |

**maintain 模式校验规则（validate）：**

与 add 的区别在于 maintain 模式要求 id 非空而非 eid/key/name/content（允许部分更新）。updatetime 自动设为当前时间。

### 响应结构

`data.result` = update 影响行数。

### 调用链

```
SmsTemplet.maintain
├─ reqjson 取 id 手动校验
├─ validate(smsTemplet, "maintain")
└─ ISmsTempletService.update(smsTemplet)
```

---

## 4. op=SmsTemplet.del — 删除短信模板

### 请求格式
{"op": "SmsTemplet.del", "action": "sms", "data": {"id": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| id | 是 | 模板主键，>= 1 |

### 响应结构

`data.result` = 删除影响行数。

### 调用链

```
SmsTemplet.del → ISmsTempletService.delete(id)
```

---

## 5. op=SmsTemplet.get — 获取短信模板详情

### 请求格式
{"op": "SmsTemplet.get", "action": "sms", "data": {"id": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| id | 是 | 模板主键，>= 1 |

### 响应结构

- 命中：`data.object` = `DbSmsTemplet` JSON。
- 未命中：`data.object` 不存在（空 map）。

### 调用链

```
SmsTemplet.get → ISmsTempletService.get(id)
```

---

相关文档：[00-分支索引](00-分支索引.md) · [service-SmsCfg](service-SmsCfg.md) · [service-SmsTask](service-SmsTask.md)
