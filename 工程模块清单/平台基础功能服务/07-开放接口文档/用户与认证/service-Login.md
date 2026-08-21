# service-Login — 登录/退出接口（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/user/Login.java`
> 基础类：`cn.testin.service.GenericBaseService`
> 机制：请求走 `ApiServlet /*` 入口，通过 `action=user` + `op=Login.方法名` 路由到此类的对应 public 方法；每个方法的参数为 `ApiRequest`，返回 JSON 字符串。
> - **action**: `user`（对应包 `cn.testin.service.user`）
> - **入口格式**：`{"op": "Login.方法名", "action": "user", "data": {...}}`
> 业务：用户登录（支持 testPro/LDAP/SSO 三种模式）、用户登出。这是登录的主链路。

## op 列表总表

| # | op | 方法名 | 说明 |
|---|---|---|---|
| 1 | Login.login | login | 用户登录（签发 sid） |
| 2 | Login.logout | logout | 用户退出（销毁 sid） |

---

## 1. op=Login.login — 用户登录

### 请求格式
{"op": "Login.login", "action": "user", "data": {"email": "...", "pwd": "...", "landingMode": "testPro"|"ldap"|"SSO", ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| email | String | 否 | 登录邮箱（`isNull` 判断，未传为 null） |
| loginMobile | String | 否 | 登录手机号 |
| pwd | String | 否 | 密码（`isNull` 判断，未传为 null） |
| channel | String | 否 | 渠道（支持 `appName` 或 `channel` 字段） |
| version | String | 否 | 客户端版本 |
| ip | String | 否 | 客户端 IP |
| expireTime | Long | 否 | token 过期时间（毫秒时间戳） |
| kick | Boolean | 否 | 是否互踢（布尔） |
| eid | Integer | 否 | 企业 ID |
| obvious | Boolean | 否 | 密码是否明文（true 时 MD5 后使用） |
| iTestinProAuth | Boolean | 否 | 是否走 itestinPro 鉴权 |
| landingMode | String | 否 | 登录模式：`"ldap"` / `"SSO"` / 默认 `"testPro"` |

### 核心逻辑

三种登录模式：

**1. testPro（默认）**：直接调用 `userService.login(email, loginMobile, pwd, ...)` 签发 sid。

**2. LDAP 模式**（`landingMode="ldap"`）：
- 加载 `config.properties` 配置 LDAP 连接参数（host、port、domain），也支持环境变量覆盖。
- 若 email 含 `@` 则直接 LDAP 认证；否则使用 `email + domain` 拼接邮箱后认证。
- LDAP 认证通过后，查找本地数据库用户并取其加密密码，调用 `userService.login()` 签发 sid。
- 若配置了 `oem_ldap_init_user` 参数且用户无密码，则自动创建初始化用户。

**3. SSO 模式**（`landingMode="SSO"`）：跳过密码校验，直接从数据库取用户加密密码后签发 sid。

### 签发后锁定检查

sid 签发成功后：
1. 检查 `status == NOT_ACTIVE`：已被管理员锁定则返回 30000 错误。
2. 检查 `oem_lock_user` 参数：超过配置天数未登录则自动锁定用户并返回 30000 错误。
3. 更新 `loginTime`，记录登录日志到 `user_login_log` 表。

### 响应

`{ error_code, msg, data: { result: sid字符串 } }` — 失败返回 `error_code=30000`（用户被锁定）。

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功；30000 用户被锁定 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | String | 登录令牌（sid），登录失败为空字符串 |

### 涉及表

- `db_user`（`DbUserInfo`）— 用户信息查询、更新 loginTime
- `db_online`（`DbUserOnline`）— 在线表写入 sid
- `db_system_param`（`DbSystemParam`）— OEM 配置参数
- `user_login_log` — 登录统计日志

---

## 2. op=Login.logout — 用户退出

### 请求格式
{"op": "Login.logout", "action": "user", "data": {"sid": "...", "ip": "...", "needAop": false}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sid | String | 是 | 登录令牌（优先从 header 取，`isBlank` 后 `optString("sid")`） |
| ip | String | 否 | IP 地址（`optString`） |
| needAop | Boolean | 否 | 是否需要 AOP 通知（`optBoolean`，默认 false） |

### 核心逻辑

调用 `userService.logout(sid, ip, needAop)` 销毁在线记录，清除 `db_online` 表中的会话。

### 响应

`{ error_code, msg, data: { result: Integer } }` — `result` 为销毁的在线记录数。

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 销毁的在线记录数 |

### 涉及表

- `db_online`（`DbUserOnline`）— 删除在线记录
