# service-Online — 用户在线信息接口（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/user/Online.java`
> 基础类：`cn.testin.service.GenericBaseService`
> 机制：请求走 `ApiServlet /*` 入口，通过 `action=user` + `op=Online.方法名` 路由到此类的对应 public 方法；每个方法的参数为 `ApiRequest`，返回 JSON 字符串。
> - **action**: `user`（对应包 `cn.testin.service.user`）
> - **入口格式**：`{"op": "Online.方法名", "action": "user", "data": {...}}`
> 业务：查询用户在线状态、在线用户列表、踢出在线用户、刷新 sid 过期时间、修改第三方鉴权信息、在线人数统计。

## op 列表总表

| # | op | 方法名 | 说明 |
|---|---|---|---|
| 1 | Online.getUserOnline | getUserOnline | 根据 sid 查询用户在线信息 |
| 2 | Online.list | list | 查询在线用户列表（分页） |
| 3 | Online.remove | remove | 断开在线用户（踢出） |
| 4 | Online.refresh | refresh | 刷新 sid 过期时间 |
| 5 | Online.modifyThirdInfo | modifyThirdInfo | 修改第三方鉴权信息（ssoAppId/authParams） |
| 6 | Online.getOnlineNums | getOnlineNums | 查询当前在线人数 |

---

## 1. op=Online.getUserOnline — 在线用户信息

### 请求格式
{"op": "Online.getUserOnline", "action": "user", "data": {"sid": "..."}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sid | String | 否 | 登录令牌（`isNull` 判断，未传为 null） |
| needApiKey | Integer | 否 | 是否返回 apiKey（`optInt` 默认 0，1 时返回） |

### 核心逻辑

查 `db_online` 获取在线用户信息，同时补查用户权限（`lastRelation`）和用户手机号。

### 响应

```json
{ error_code, msg, data: { object: {
  eid, projectid, pname_pro, userid, sid, email, mobile, name,
  channel, version, expireTime, cc, loginMobile, roleId, authParams, ssoAppId,
  (needApiKey=1 时) apiKey
} } }
```

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 在线用户信息（不存在时为空对象） |
| data.object.eid | Integer | 企业 ID |
| data.object.projectid | Integer | 项目组 ID |
| data.object.pname_pro | String | 项目名称 |
| data.object.userid | Integer | 用户 ID |
| data.object.sid | String | 登录令牌 |
| data.object.email | String | 邮箱 |
| data.object.mobile | String | 手机号 |
| data.object.name | String | 用户姓名 |
| data.object.channel | String | 渠道 |
| data.object.version | String | 客户端版本 |
| data.object.expireTime | Long | 过期时间 |
| data.object.cc | String | 国家区号 |
| data.object.loginMobile | String | 登录手机号 |
| data.object.roleId | Integer | 角色 ID |
| data.object.authParams | String | 鉴权参数 |
| data.object.ssoAppId | String | SSO 应用 ID |
| data.object.apiKey | String | API Key（needApiKey=1 时返回） |

---

## 2. op=Online.list — 在线用户列表

### 请求格式
{"op": "Online.list", "action": "user", "data": {"email": "...", "page": ..., "pageSize": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| email | String | 否 | 邮箱筛选（`isNull` 判断） |
| userName | String | 否 | 姓名筛选 |
| page | Integer | 否 | 页码 |
| pageSize | Integer | 否 | 每页条数 |

### 核心逻辑

调用 `userService.getUserStatistics()` 分页查询在线用户。

### 响应

`{ error_code, msg, data: { list: [ { sid, email, name }, ... ], page, pageSize, totalRow, totalPage } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 在线用户列表 |
| data.list[].sid | String | 登录令牌 |
| data.list[].email | String | 邮箱 |
| data.list[].name | String | 用户姓名 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页条数 |
| data.totalRow | Integer | 总记录数 |
| data.totalPage | Integer | 总页数 |

---

## 3. op=Online.remove — 踢出在线用户

### 请求格式
{"op": "Online.remove", "action": "user", "data": {"sid": "..."}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sid | String | 否 | 登录令牌（`isNull` 判断，未传为 null） |

### 核心逻辑

调用 `userService.removeUserStatistics(sid)` 删除在线记录。删除 0 行返回 -1 错误。

### 响应

`{ error_code, msg, data: { result: rowNum } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功；-1 删除失败 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 删除的在线记录数 |

---

## 4. op=Online.refresh — 刷新 sid 过期时间

### 请求格式
{"op": "Online.refresh", "action": "user", "data": {"sid": "...", "expirePeriod": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sid | String | 是 | 登录令牌（`isBlank` 判空，为空抛参数异常） |
| expirePeriod | Long | 否 | 续期时长（毫秒，默认 864000000 = 10 天） |

### 核心逻辑

校验 sid 有效性且未过期后，更新 `expireTime = expirePeriod + now`。若 sid 已过期则清理在线记录并报错。

### 响应

`{ error_code, msg, data: { result: 1(成功) / 0(失败) } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 1 成功 / 0 失败 |

---

## 5. op=Online.modifyThirdInfo — 修改第三方鉴权信息

### 请求格式
{"op": "Online.modifyThirdInfo", "action": "user", "data": {"sid": "...", "ssoAppId": "...", "authParams": "..."}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sid | String | 是 | 登录令牌（`isBlank` 判空） |
| ssoAppId | String | 是 | SSO 应用 ID（`isNull` 判空） |
| authParams | String | 是 | 鉴权参数（`isNull` 判空） |

### 核心逻辑

校验 sid 有效且未过期后，更新 `ssoAppId` 和 `authParams` 字段。

### 响应

`{ error_code, msg, data: { result: 1(成功) / 0(失败) } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 1 成功 / 0 失败 |

---

## 6. op=Online.getOnlineNums — 在线人数

### 请求格式
{"op": "Online.getOnlineNums", "action": "user", "data": {}}

### 请求参数

无。

### 核心逻辑

统计 `db_online` 中 `expire_time > 当前时间` 的去重 user_id 数量。

### 响应

`{ error_code, msg, data: { result: 在线人数 } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 当前在线人数 |

---

### 涉及表

- `db_online`（`DbUserOnline`）— 在线状态表
- `db_enterprise_user_relation`（`DbEnterpriseUserRelation`）— 用于补查用户权限 roleId
