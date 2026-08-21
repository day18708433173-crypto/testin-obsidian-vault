# service-EmailCfg — 邮件配置

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/email/EmailCfg.java`
> 类：`cn.testin.service.email.EmailCfg extends GenericBaseService`（非 Spring MVC Controller）
> 入口方式：ApiServlet 网关按 `action` 定位（`email.EmailCfg`），`op` 反射调用方法名；`ApiRequest.reqjson` 传参；返回 JSON 字符串。
> - **action**: `email`（对应包 `cn.testin.service.email`）
> - **入口格式**：`{"op": "EmailCfg.方法名", "action": "email", "data": {...}}`
> 依赖：`IEmailCfgService`（继承自 `GenericBaseService`）
> 业务：发件服务器配置（SMTP 邮箱/密码/主机/端口/SSL）的增删改查与测试发送。一个邮件配置对应一个发件人身份。

## 方法列表总表

| # | op | 方法 | 说明 |
|---|---|---|---|
| 1 | EmailCfg.list | list | 邮件配置分页列表 |
| 2 | EmailCfg.get | get | 按 id 获取单条邮件配置 |
| 3 | EmailCfg.maintain | maintain | 更新邮件配置 |
| 4 | EmailCfg.add | add | 新增邮件配置 |
| 5 | EmailCfg.del | del | 按 id 删除邮件配置 |
| 6 | EmailCfg.testSendEmail | testSendEmail | 测试邮件发送（不落库，实时发送验证） |

统一返回：JSON 字符串结构 `{ code, msg, data }`。分页由 `baseListToResData` 转换。参数校验失败返回 `paraInvalid`。

公共分页规则：`page < 1` 归 1；`pageSize` 超限取 `Config.MaxSize`。

---

## 1. op=EmailCfg.list — 邮件配置分页列表

### 请求格式
{"op": "EmailCfg.list", "action": "email", "data": {"page": ..., "pageSize": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| page | 否 | 页码，默认 1 |
| pageSize | 否 | 每页条数，超限取 MaxSize |

### 响应结构

`data` 由 `baseListToResData` 从 `BaseList<DbEmailCfg>` 转换的分页结构。

### 实现意图

无条件分页查询所有邮件配置，conditionMap 为空。

### 调用链

```
EmailCfg.list → IEmailCfgService.list(conditionMap, page, pageSize)
```

---

## 2. op=EmailCfg.get — 获取单条邮件配置

### 请求格式
{"op": "EmailCfg.get", "action": "email", "data": {"id": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| id | 是 | 配置主键，必须 > 0 |

### 响应结构

`data.object` = `DbEmailCfg` JSON 对象。

### 调用链

```
EmailCfg.get → IEmailCfgService.get(id)
```

---

## 3. op=EmailCfg.maintain — 更新邮件配置

### 请求格式
{"op": "EmailCfg.maintain", "action": "email", "data": {"id": ..., "email": "...", ...}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Integer | 是 | 配置主键，必须 > 0 |
| type | Integer | 否 | 类型，默认 0 |
| email | String | 否 | 发件邮箱地址 |
| pwd | String | 否 | 邮箱密码/授权码 |
| nickName | String | 否 | 发件人昵称 |
| host | String | 否 | SMTP 主机地址 |
| port | Integer | 否 | SMTP 端口 |
| ssl | Integer | 否 | 是否启用 SSL，0 或 1 |

### 响应结构

`data.result` = update 影响行数。

### 实现意图

按 id 更新发件服务器配置；所有字段可选，只更新传入的字段。

### 调用链

```
EmailCfg.maintain → IEmailCfgService.update(emailCfg)
```

---

## 4. op=EmailCfg.add — 新增邮件配置

### 请求格式
{"op": "EmailCfg.add", "action": "email", "data": {"email": "...", "pwd": "...", "nickName": "...", "host": "...", "port": ..., "ssl": 0|1}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| email | String | 是 | 发件邮箱地址 |
| pwd | String | 是 | 邮箱密码/授权码 |
| nickName | String | 是 | 发件人昵称 |
| host | String | 是 | SMTP 主机地址 |
| port | Integer | 是 | SMTP 端口，必须 > 0 |
| ssl | Integer | 是 | 是否启用 SSL，0 或 1 |
| type | Integer | 否 | 类型，默认 0 |

### 响应结构

`data.result` = insert 条数。

### 调用链

```
EmailCfg.add → IEmailCfgService.add(emailCfg)
```

---

## 5. op=EmailCfg.del — 删除邮件配置

### 请求格式
{"op": "EmailCfg.del", "action": "email", "data": {"id": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| id | 是 | 配置主键，必须 > 0 |

### 响应结构

`data.result` = 删除影响行数。

### 调用链

```
EmailCfg.del → IEmailCfgService.delete(id)
```

---

## 6. op=EmailCfg.testSendEmail — 测试邮件发送

### 请求格式
{"op": "EmailCfg.testSendEmail", "action": "email", "data": {"emailCfg": {...}, "emailTask": {...}}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| emailCfg | JSONObject | 是 | 发件服务器配置对象（含 email/pwd/host/port/ssl/type） |
| emailTask | JSONObject | 是 | 邮件内容对象（含 to/type、可选 subject/content） |

### 校验（emailCfg）

| 字段 | 校验 |
|---|---|
| email | 不能为空 |
| pwd | 不能为空 |
| host | 不能为空 |
| port | 不能为空，必须 > 0 且 <= 65535 |
| ssl | 不能为空 |
| type | 不能为空 |

### 校验（emailTask）

| 字段 | 校验 |
|---|---|
| to | 不能为空 |
| type | 不能为空 |

### 响应结构

`data.result` = `boolean`（发送成功/失败）。

### 实现意图

不落库实时发送一封测试邮件，用于验证发件服务器配置是否正确。emailCfg 和 emailTask 均由 `Config.gson.fromJson` 反序列化，校验后交由 `IEmailCfgService.testSendEmail` 执行发送。

### 调用链

```
EmailCfg.testSendEmail → IEmailCfgService.testSendEmail(emailCfg, emailTask)
```

---

相关文档：[00-分支索引](00-分支索引.md) · [service-EmailTask](service-EmailTask.md) · [service-EmailTempletCfg](service-EmailTempletCfg.md)
