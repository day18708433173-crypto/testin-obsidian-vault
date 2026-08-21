# UserController — 用户控制器（辅助接口）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/UserController.java`
> 类级路由：`/user`
> Service 实现：`cn.testin.business.impl.user.UserService`、`cn.testin.business.interfaces.user.IRoleService`、`cn.testin.business.interfaces.user.IUserLoginLogService`、`cn.testin.business.interfaces.user.ISystemService`
> 业务：用户登录（辅助链路）、角色功能查询、功能开关、用户统计、用户列表、用户信息修改、角色列表。
>
> **注意**：真正的登录主链路走 `ApiServlet` 入口，对应 `cn.testin.service.user.Login.login`。此 MVC 控制器里的 `/v3/user/users/login` 是辅助通道（含 SSO 分支、验证码校验、锁定状态检查和 Cookie 下发）。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|---|---|---|---|
| 1 | GET | `/v3/user/role/functions` | getRoleFunctions | 通过角色 ID 获取角色基础功能列表 |
| 2 | GET | `/v3/user/functions/switches` | getFunctionsSwitch | 获取功能开关列表（分页，默认 100 条/页） |
| 3 | GET | `/v3/user/login/statistic` | loginStatistic | 用户活跃统计（按企业/时间段分页查询） |
| 4 | POST | `/v3/user/get_user_list` | getUserList | 根据 userId 列表批量查用户 |
| 5 | PUT | `/v3/user/modify_user_info` | updateUser | 修改用户信息（强制置 null 掉 roleId） |
| 6 | GET | `/v3/user/roles` | getRoleList | 查询企业角色列表 |
| 7 | POST | `/v3/user/users/login` | login | 辅助登录入口（含 SSO、验证码校验、锁定检查） |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`。

涉及表：`db_user` 系列表（`DbUserInfo`、`DbRoleInfo`、`ViewEnterpriseUser` 等），底层通过 `UserService` / `IUserLoginLogService` / `IRoleService` 操作。

---

## 1. GET /v3/user/role/functions — 获取角色基础功能

### 入口

`UserController.getRoleFunctions(@RequestParam("role_id") Integer roleId)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| role_id | Integer | 是 | 角色 ID |

### 响应结构

`ResponseResult<BaseListResponseDTO<UserRoleFunctionsResponseDTO>>`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 角色功能列表（UserRoleFunctionsResponseDTO） |
| data.list[].configKey | String | 配置键 |
| data.list[].displayName | String | 显示名称 |
| data.list[].funId | Integer | 功能 ID |
| data.list[].href | String | 链接地址 |
| data.list[].parentId | Integer | 父功能 ID |
| data.list[].roleId | Integer | 角色 ID |

### 实现意图

根据角色 ID 查询该角色拥有的基础功能集合，返回功能列表。

### 调用链

```
UserController.getRoleFunctions
└─ IRoleService.getRoleFunctionsByRoleId
```

---

## 2. GET /v3/user/functions/switches — 获取功能开关配置

### 入口

`UserController.getFunctionsSwitch(page, pageSize)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| page | Integer | 否 | 1 | 页码 |
| page_size | Integer | 否 | 100 | 每页条数 |

### 响应结构

`ResponseResult<BaseListResponseDTO<FunctionSwitchConfigResponseDTO>>`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 功能开关配置列表（FunctionSwitchConfigResponseDTO） |
| data.list[].description | String | 功能描述 |
| data.list[].key | String | 配置键 |
| data.list[].type | Integer | 类型 |

### 实现意图

分页查询系统功能开关配置列表，后台可管理各功能模块的启用/停用。

### 调用链

```
UserController.getFunctionsSwitch
└─ IRoleService.getFunctionsSwitch
```

---

## 3. GET /v3/user/login/statistic — 用户活跃统计

### 入口

`UserController.loginStatistic(eid, startTime, endTime, page, pageSize)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| eid | Integer | 否 | 1 | 企业 ID |
| start_time | Long | 否 | — | 开始时间戳（毫秒） |
| end_time | Long | 否 | — | 结束时间戳（毫秒） |
| page | Integer | 否 | 1 | 页码 |
| page_size | Integer | 否 | 20 | 每页条数 |

### 响应结构

`ResponseResult<BasePageListResponseDTO<UserCountResponseDTO>>`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 用户登录统计列表（UserCountResponseDTO） |
| data.list[].userCount | Long | 用户登录统计次数 |
| data.list[].userId | Long | 用户 ID |
| data.list[].email | String | 用户邮箱 |
| data.page / pageSize / totalPage / totalRow | Integer | 分页信息 |

### 实现意图

按企业维度统计用户登录活跃数据，支持时间段过滤和分页。

### 调用链

```
UserController.loginStatistic
└─ IUserLoginLogService.loginStatistic
```

---

## 4. POST /v3/user/get_user_list — 批量查询用户

### 入口

`UserController.getUserList(@RequestBody List<Integer> userIdList)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| (数组体) | List\<Integer\> | 是 | userId 整数列表 |

### 响应结构

`ResponseResult<List<UserInfo>>`（`cn.testin.api.bean.UserInfo`）

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Array | 用户列表（UserInfo） |
| data[].id | Integer | 用户 ID |
| data[].email | String | 登录邮箱 |
| data[].name | String | 姓名 |
| data[].mobile | String | 手机号 |
| data[].job | String | 职务 |
| data[].status | Integer | 状态：0=停用，1=正常，2=未激活 |
| data[].createTime | Long | 创建时间 |
| data[].updateTime | Long | 更新时间 |

### 实现意图

根据一组 userId 批量查询用户信息（`UserInfo` 对象）。

### 调用链

```
UserController.getUserList
└─ UserService.getUserListByUserIdList
```

---

## 5. PUT /v3/user/modify_user_info — 修改用户信息

### 入口

`UserController.updateUser(@RequestBody UpdateUserInfoDTO updateUserInfoDTO)`

### 请求参数（JSON Body，UpdateUserInfoDTO）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| userName | String | 否 | 用户名 |
| oldPassWord | String | 否 | 老密码 |
| newPassWord | String | 否 | 新密码 |
| userMobile | String | 否 | 手机号 |
| userJob | String | 否 | 用户职位 |
| roleId | Integer | 否 | 用户角色（方法内部强制置 null，即此接口不修改角色） |
| eid | Integer | 否 | 企业 ID（继承 BaseRequestDTO） |
| projectId | Integer | 否 | 项目 ID（继承 BaseRequestDTO） |
| userId | Integer | 否 | 用户 ID（继承 BaseRequestDTO） |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Long | 影响行数 |

### 实现意图

更新用户基本信息（不含角色字段）。

### 调用链

```
UserController.updateUser
└─ UserService.updateUser
```

---

## 6. GET /v3/user/roles — 查询角色列表

### 入口

`UserController.getRoleList(@RequestParam("eid") Integer eid)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业 ID |

### 响应结构

`ResponseResult<List<RoleResponseDTO>>`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Array | 角色列表（RoleResponseDTO） |
| data[].id | Integer | 角色 ID |
| data[].name | String | 角色名称 |

### 实现意图

查询指定企业下的所有角色，转换为 `RoleResponseDTO` 列表返回。

### 调用链

```
UserController.getRoleList
└─ IRoleService.getRoleList
```

---

## 7. POST /v3/user/users/login — 辅助登录入口

### 入口

`UserController.login(@RequestBody LoginAddRequestDTO requestDTO, HttpServletRequest request, HttpServletResponse response)`

### 请求参数（JSON Body，LoginAddRequestDTO）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| email | String | 是 | 登录邮箱 |
| pwd | String | 是 | 密码（obvious=true 时是明文，方法内部 MD5） |
| landingMode | String | 否 | 登录模式：`"SSO"` 走 SSO 免密；默认 `"testPro"` 走验证码 |
| loginMobile | String | 否 | 登录手机号 |
| userId | Integer | 否 | 登录用户 ID |
| version | String | 否 | 客户端版本 |
| ip | String | 否 | 客户端 IP |
| expireTime | Long | 否 | token 过期时间 |
| kick | Boolean | 否 | 是否互踢 |
| eid | Integer | 否 | 企业 ID |
| apiKey | String | 否 | API Key |
| appName | String | 否 | 应用名称 |
| channel | String | 否 | 渠道 |
| obvious | Boolean | 否 | 密码是否为明文（true 则 MD5 后使用） |
| verifyCodeId | String | 否 | 验证码 ID（非 SSO 模式必传） |
| verifyCode | String | 否 | 验证码（非 SSO 模式必传） |

### 响应结构

`ResponseEntity<String>`，body 为 `JsonUtil.toJsonString(ResponseResult<BaseResultDTO>)`，`data.result` = sid 令牌串；同时设置 `authtoken_pro` Cookie。

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功；30000 用户被锁定 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | String | 登录令牌（sid），失败为空字符串 |

### 实现意图

辅助登录通道：
1. `landingMode=SSO`：根据 email 查出 userInfo，取库中加密密码直接调用 `userService.login()` 签发 sid。
2. 默认模式：校验验证码后调用 `userService.login()` 签发 sid。
3. 签发 sid 后执行锁定检查：status=NOT_ACTIVE 直接拒绝；超时限天数未登录则自动锁定。
4. 写入 Cookie `authtoken_pro`，记录登录日志，更新 loginTime。

### 关联横切

- Cookie 域名来自 `Config.COOKIE_DOMAIN`，过期时间 `Config.COOKIE_MAX_AGE`。
- 锁定天数来自 `ISystemService.getParam("enterprise-info", "oem_lock_user")`。
