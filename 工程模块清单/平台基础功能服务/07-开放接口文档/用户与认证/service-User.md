# service-User — 用户信息接口（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/user/User.java`
> 基础类：`cn.testin.service.GenericBaseService`
> 机制：请求走 `ApiServlet /*` 入口（非 `/v3`），通过 `action=user` + `op=User.方法名` 路由到此类的对应 public 方法；每个方法的参数为 `ApiRequest`，返回 JSON 字符串。
> - **action**: `user`（对应包 `cn.testin.service.user`）
> - **入口格式**：`{"op": "User.方法名", "action": "user", "data": {...}}`
> 业务：用户 CRUD、密码校验/重置、用户列表查询、角色关系变更等。

## op 列表总表

| # | op | 方法名 | 说明 |
|---|---|---|---|
| 1 | User.addUser | addUser | 新增用户（注册） |
| 2 | User.modifyUserInfo | modifyUserInfo | 修改用户信息 |
| 3 | User.modifyStatus | modifyStatus | 修改用户状态（含解锁重置 loginTime） |
| 4 | User.modifyAccount | modifyAccount | 修改登录邮箱/手机号（绑定/替换/解绑） |
| 5 | User.checkPwd | checkPwd | 校验密码是否正确 |
| 6 | User.resetPwd | resetPwd | 重置密码 |
| 7 | User.getUser | getUser | 查询单个用户 |
| 8 | User.getEidsByEmail | getEidsByEmail | 根据邮箱查所属企业 ID 列表 |
| 9 | User.getUserList | getUserList | 分页查询用户列表（支持多条件筛选） |
| 10 | User.getProjectUserList | getProjectUserList | 查询项目组下的用户列表 |
| 11 | User.getAllUser | getAllUser | 查询所有用户（含企业名等扩展信息，分页） |
| 12 | User.getAllUserInfo | getAllUserInfo | 查询所有用户基本信息（可选含密码，分页） |
| 13 | User.updateToCREW | updateToCREW | 将角色下用户批量降级为 CREW 并更新默认角色 |
| 14 | User.updateUserDefaultRole | updateUserDefaultRole | 更新用户在企业下的默认角色 |
| 15 | User.getDefaultRoleByEidAndUid | getDefaultRoleByEidAndUid | 根据 eid+uid 获取默认角色 |
| 16 | User.getDefaultRoleUserByRoleId | getDefaultRoleUserByRoleId | 根据角色 ID 获取默认角色下的用户列表 |
| 17 | User.getProjectRoleInfo | getProjectRoleInfo | 根据 eid+uid 查询用户的项目角色信息 |
| 18 | User.getAdUsersByNameOrMail | getAdUsersByNameOrMail | 从 AD 域搜索用户 |
| 19 | User.queryAll | queryAll | 根据 sid 获取用户全部关联信息（用户+企业+项目+角色+项目管理员） |
| 20 | User.getToken | getToken | 根据当前在线用户信息签发新 sid |

---

## 1. op=User.addUser — 新增用户

### 请求格式
{"op": "User.addUser", "action": "user", "data": {"userInfo": {...}}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| userInfo | Object | 是 | JSON 对象（`optString` 解析为 DbUserInfo） |
| userInfo.email | String | 是 | 邮箱（`isBlank` 判空） |
| userInfo.pwd | String | 是 | 密码（`isEmpty` 判空） |
| userInfo.name | String | 否 | 姓名 |
| userInfo.mobile | String | 否 | 手机号 |

### 核心逻辑

解析 `userInfo` 为 `DbUserInfo`，调用 `userService.addUser()` 写入 `db_user` 表。遇唯一键冲突（email 重复）返回 `USER_EMAIL_EXIST_ERROR`。

### 响应

`{ error_code, msg, data: { result } }` — `result` 为新 userId。

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 新用户 ID |

---

## 2. op=User.modifyUserInfo — 修改用户信息

### 请求格式
{"op": "User.modifyUserInfo", "action": "user", "data": {"userInfo": {...}}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| userInfo | Object | 是 | JSON 对象，解析为 DbUserInfo |
| userInfo.id | Integer | 是 | 用户 ID（`< 1` 返回参数错误） |
| userInfo.其他字段 | — | 否 | 修改字段按需传入 |

### 核心逻辑

调用 `userService.modifyUserInfo(userInfo)` 更新 `db_user`。

### 响应

`{ error_code, msg, data: { result: boolean } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Boolean | 是否修改成功 |

---

## 3. op=User.modifyStatus — 修改用户状态

### 请求格式
{"op": "User.modifyStatus", "action": "user", "data": {"userInfo": {...}}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| userInfo | Object | 是 | JSON 对象，解析为 DbUserInfo |
| userInfo.id | Integer | 是 | 用户 ID |
| userInfo.status | Integer | 是 | 目标状态（1=激活，会重置 loginTime=0） |

### 核心逻辑

与 `modifyUserInfo` 类似，但额外处理：若目标 `status=1`（激活），则将 `loginTime` 置 0 以解锁超时锁定。

### 响应

`{ error_code, msg, data: { result: boolean } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Boolean | 是否修改成功 |

---

## 4. op=User.modifyAccount — 修改登录账号

### 请求格式
{"op": "User.modifyAccount", "action": "user", "data": {"userid": ..., "op": 1|2, ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| userid | Integer | 否 | 用户 ID（`getInt`） |
| op | Integer | 否 | 操作类型：1=绑定/替换，2=解绑手机号 |
| email | String | 否 | 绑定/替换时的邮箱 |
| cc | String | 否 | 国家区号 |
| loginMobile | String | 否 | 绑定/替换时的手机号 |

### 核心逻辑

调用 `userService.modifyAccount(userid, op, email, cc, loginMobile)` 绑定、换绑或解绑登录账号。

### 响应

`{ error_code, msg, data: { result: boolean } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Boolean | 是否操作成功 |

---

## 5. op=User.checkPwd — 校验密码

### 请求格式
{"op": "User.checkPwd", "action": "user", "data": {"userid"|"email": ..., "pwd"|"encryptedPwd": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| userid | Integer | 否 | 用户 ID（与 email 二选一） |
| email | String | 否 | 邮箱（与 userid 二选一） |
| pwd | String | 否 | 明文密码（与 encryptedPwd 二选一，`isBlank` 判空） |
| encryptedPwd | String | 否 | 加密后的密码（与 pwd 二选一） |

### 核心逻辑

根据 userid 或 email 查用户，将传入的明文（MD5 后）或已加密密码与库中密码比对。

### 响应

`{ error_code, msg, data: { result: 1(匹配) / 0(不匹配) } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 1 匹配 / 0 不匹配 |

---

## 6. op=User.resetPwd — 重置密码

### 请求格式
{"op": "User.resetPwd", "action": "user", "data": {"userid"|"email": ..., "pwd"|"encryptedPwd": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| userid | Integer | 否 | 用户 ID（与 email 二选一） |
| email | String | 否 | 邮箱（与 userid 二选一） |
| pwd | String | 否 | 新明文密码（与 encryptedPwd 二选一） |
| encryptedPwd | String | 否 | 新加密密码（与 pwd 二选一） |

### 核心逻辑

根据 userid 或 email 找到用户，更新 `db_user.pwd` 字段。传入 `encryptedPwd` 直接写入，否则 MD5 后写入。

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

## 7. op=User.getUser — 查单个用户

### 请求格式
{"op": "User.getUser", "action": "user", "data": {"userid"|"email"|"loginMobile"|"sid"|"thirdPartyUserid": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| userid | Integer | 否 | 用户 ID（多选一，须 > 0） |
| email | String | 否 | 邮箱（多选一） |
| loginMobile | String | 否 | 登录手机号（多选一） |
| sid | String | 否 | 令牌（多选一） |
| eid | Integer | 否 | 企业 ID |
| thirdPartyUserid | String | 否 | 第三方用户 ID（多选一） |

### 核心逻辑

调用 `userService.getUser()` 查询单个用户。支持通过 `thirdPartyUserid` 先查用户 ID 再获取详情。

### 响应

`{ error_code, msg, data: { object: { userid, email, name, mobile, job, status, createTime, cc, loginMobile, thirdPartyUserid } } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 用户对象（不存在时为空对象） |
| data.object.userid | Integer | 用户 ID |
| data.object.email | String | 邮箱 |
| data.object.name | String | 姓名 |
| data.object.mobile | String | 手机号 |
| data.object.job | String | 职位 |
| data.object.status | Integer | 状态 |
| data.object.createTime | Long | 创建时间 |
| data.object.cc | String | 国家区号 |
| data.object.loginMobile | String | 登录手机号 |
| data.object.thirdPartyUserid | String | 第三方用户 ID |

---

## 8. op=User.getEidsByEmail — 根据邮箱查企业 ID 列表

### 请求格式
{"op": "User.getEidsByEmail", "action": "user", "data": {"email": "..."}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| email | String | 是 | 邮箱（`isBlank` 判空，为空返回参数错误） |

### 核心逻辑

调用 `userService.getEidsByEmail(email)` 查询用户所属的企业 ID 数组。

### 响应

`{ error_code, msg, data: { object: { eids: Integer[] } } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 结果对象 |
| data.object.eids | Array | 企业 ID 数组（Integer[]） |

---

## 9. op=User.getUserList — 用户列表（分页）

### 请求格式
{"op": "User.getUserList", "action": "user", "data": {"eid": ..., "keywords": "...", ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID |
| keywords | String | 否 | 搜索关键字 |
| roleId | Integer | 否 | 角色筛选 |
| statusList | Array | 否 | 状态筛选（JSON 数组，Integer[]） |
| useridList | Array | 否 | 用户 ID 列表筛选（Integer[]） |
| isEnterpriseUser | Integer | 否 | 1=仅查企业下用户 |
| excludeAdminRole | Integer | 否 | 1=过滤系统管理员 |
| page | Integer | 否 | 页码 |
| pageSize | Integer | 否 | 每页条数 |

### 核心逻辑

调用 `userService.getUserList()` 分页查询 `view_enterprise_user`，返回分页数据。

### 响应

`{ error_code, msg, data: { list: [...], page, pageSize, totalRow, totalPage } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 用户列表 |
| data.list[].userid | Integer | 用户 ID |
| data.list[].email | String | 邮箱 |
| data.list[].name | String | 姓名 |
| data.list[].mobile | String | 手机号 |
| data.list[].job | String | 职位 |
| data.list[].status | Integer | 状态 |
| data.list[].createTime | Long | 创建时间 |
| data.list[].roleid | Integer | 角色 ID |
| data.list[].cc | String | 国家区号 |
| data.list[].loginMobile | String | 登录手机号 |
| data.list[].lockStatus | Integer | 锁定状态 |
| data.list[].thirdPartyUserid | String | 第三方用户 ID |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页条数 |
| data.totalRow | Integer | 总记录数 |
| data.totalPage | Integer | 总页数 |

---

## 10. op=User.getProjectUserList — 项目组用户列表

### 请求格式
{"op": "User.getProjectUserList", "action": "user", "data": {"eid": ..., "projectid": ..., ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID |
| projectid | Integer | 否 | 项目组 ID |
| roleidList | Array | 否 | 角色 ID 列表（Integer[]） |
| keywords | String | 否 | 搜索关键字 |
| status | Integer | 否 | 状态 |
| page | Integer | 否 | 页码 |
| pageSize | Integer | 否 | 每页条数 |

### 核心逻辑

调用 `userService.getProjectUserList()` 查询项目组视图下的用户。

### 响应

`{ error_code, msg, data: { list: [...], page, pageSize, totalRow, totalPage } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 用户列表 |
| data.list[].projectid | Integer | 项目组 ID |
| data.list[].userid | Integer | 用户 ID |
| data.list[].eid | Integer | 企业 ID |
| data.list[].email | String | 邮箱 |
| data.list[].name | String | 姓名 |
| data.list[].mobile | String | 手机号 |
| data.list[].job | String | 职位 |
| data.list[].status | Integer | 状态 |
| data.list[].createTime | Long | 创建时间 |
| data.list[].roleid | Integer | 角色 ID |
| data.list[].cc | String | 国家区号 |
| data.list[].loginMobile | String | 登录手机号 |
| data.list[].thirdPartyUserid | String | 第三方用户 ID |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页条数 |
| data.totalRow | Integer | 总记录数 |
| data.totalPage | Integer | 总页数 |

---

## 11. op=User.getAllUser — 查询所有用户（含企业名）

### 请求格式
{"op": "User.getAllUser", "action": "user", "data": {"enterpriseName": "...", "page": ..., ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| enterpriseName | String | 否 | 企业名筛选 |
| email | String | 否 | 邮箱筛选 |
| mobile | String | 否 | 手机号筛选 |
| job | String | 否 | 职位筛选 |
| userStatus | Integer | 否 | 状态筛选 |
| userid | Integer | 否 | 用户 ID 筛选 |
| eid | Integer | 否 | 企业 ID 筛选 |
| page | Integer | 否 | 页码 |
| pageSize | Integer | 否 | 每页条数 |

### 核心逻辑

调用 `userService.getAllUser(params)` 多条件分页查询。

### 响应

`{ error_code, msg, data: { list: [...], page, pageSize, totalRow, totalPage } }` — 列表中每条含 `enterpriseFullName`。

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 用户列表 |
| data.list[].fullName | String | 企业全名 |
| data.list[].eid | Integer | 企业 ID |
| data.list[].userid | Integer | 用户 ID |
| data.list[].email | String | 邮箱 |
| data.list[].name | String | 姓名 |
| data.list[].mobile | String | 手机号 |
| data.list[].job | String | 职位 |
| data.list[].status | Integer | 状态 |
| data.list[].createTime | Long | 创建时间 |
| data.list[].roleid | Integer | 角色 ID |
| data.list[].cc | String | 国家区号 |
| data.list[].loginMobile | String | 登录手机号 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页条数 |
| data.totalRow | Integer | 总记录数 |
| data.totalPage | Integer | 总页数 |

---

## 12. op=User.getAllUserInfo — 查询所有用户基本信息

### 请求格式
{"op": "User.getAllUserInfo", "action": "user", "data": {"useridList": [...], ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| useridList | Array | 否 | ID 列表（Integer[]） |
| keywords / email | String | 否 | 关键字/邮箱筛选 |
| name | String | 否 | 姓名筛选 |
| status | Integer | 否 | 状态筛选 |
| hasPwd | Integer | 否 | 是否返回密码字段（>0 则返回） |
| page | Integer | 否 | 页码 |
| pageSize | Integer | 否 | 每页条数 |

### 核心逻辑

调用 `userService.getUserInfoList()` 分页查询 `db_user`。

### 响应

`{ error_code, msg, data: { list: [...], page, pageSize, totalRow, totalPage } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 用户列表 |
| data.list[].userid | Integer | 用户 ID |
| data.list[].email | String | 邮箱 |
| data.list[].name | String | 姓名 |
| data.list[].mobile | String | 手机号 |
| data.list[].job | String | 职位 |
| data.list[].status | Integer | 状态 |
| data.list[].createTime | Long | 创建时间 |
| data.list[].pwd | String | 密码（hasPwd>0 时返回） |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页条数 |
| data.totalRow | Integer | 总记录数 |
| data.totalPage | Integer | 总页数 |

---

## 13. op=User.updateToCREW — 角色降级为 CREW

### 请求格式
{"op": "User.updateToCREW", "action": "user", "data": {"roleId": ..., "eid": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| roleId | Integer | 是 | 原角色 ID（`isNull` 判空；必须 > CREW 值） |
| eid | Integer | 是 | 企业 ID（`isNull` 判空） |

### 核心逻辑

将该角色下的所有用户批量降级为 `CREW`，同时更新默认角色关系。

### 响应

`{ error_code, msg, data: { result } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 更新结果 |

---

## 14. op=User.updateUserDefaultRole — 更新用户默认角色

### 请求格式
{"op": "User.updateUserDefaultRole", "action": "user", "data": {"eid": ..., "roleId": ..., "userId": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业 ID（`isNull` 判空） |
| roleId | Integer | 是 | 新角色 ID（`isNull` 判空） |
| userId | Integer | 是 | 用户 ID（`isNull` 判空） |

### 核心逻辑

更新用户 `db_user_role` 中的默认角色。

### 响应

`{ error_code, msg, data: { result } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 更新结果 |

---

## 15. op=User.getDefaultRoleByEidAndUid — 获取默认角色

### 请求格式
{"op": "User.getDefaultRoleByEidAndUid", "action": "user", "data": {"eid": ..., "userId": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业 ID（null 返回参数错误） |
| userId | Integer | 是 | 用户 ID（null 返回参数错误） |

### 响应

`{ error_code, msg, data: { object: DbUserRoleInfo } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 用户默认角色对象（DbUserRoleInfo，字段代码未确认） |

---

## 16. op=User.getDefaultRoleUserByRoleId — 按角色查用户

### 请求格式
{"op": "User.getDefaultRoleUserByRoleId", "action": "user", "data": {"eid": ..., "roleId": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业 ID（null 返回参数错误） |
| roleId | Integer | 是 | 角色 ID（null 返回参数错误） |

### 响应

`{ error_code, msg, data: { list: [DbUserRoleInfo, ...] } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 用户默认角色列表（元素为 DbUserRoleInfo，字段代码未确认） |

---

## 17. op=User.getProjectRoleInfo — 用户项目角色信息

### 请求格式
{"op": "User.getProjectRoleInfo", "action": "user", "data": {"eid": ..., "userId": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业 ID（null 返回参数错误） |
| userId | Integer | 是 | 用户 ID（null 返回参数错误） |

### 响应

`{ error_code, msg, data: { list: [DbUserProjectRoleInfo, ...] } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 用户项目角色列表（元素为 DbUserProjectRoleInfo，字段代码未确认） |

---

## 18. op=User.getAdUsersByNameOrMail — AD 域搜索用户

### 请求格式
{"op": "User.getAdUsersByNameOrMail", "action": "user", "data": {"name": "..."}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| name | String | 是 | 姓名或邮箱搜索关键字（getString） |

### 核心逻辑

调用 AD 域接口按名称或邮箱搜索用户。

### 响应

`{ error_code, msg, data: { list: [{...}, ...] } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | AD 用户列表（元素为 Map，字段代码未确认） |

---

## 19. op=User.queryAll — 查询用户全部关联信息

### 请求格式
{"op": "User.queryAll", "action": "user", "data": {"sid": "..."}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sid | String | 是 | 用户令牌（null 返回参数错误） |

### 核心逻辑

根据 sid 查询 `ViewUserEnterpriseProjectRole`（用户+企业+项目+角色的聚合视图），并额外查询当前项目的管理员列表。

### 响应

`{ error_code, msg, data: { userid, eid, projectid, userEmail, ..., projectAdmin: [ {userId, userName, userEmail} ] }, apikey, mkey }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| apikey | String | API Key |
| mkey | String | 网关模块标识 |
| data | Object | 用户-企业-项目-角色聚合对象（ViewUserEnterpriseProjectRole，字段代码未确认） |
| data.projectAdmin | Array | 项目管理员列表 |
| data.projectAdmin[].userId | Integer | 管理员用户 ID |
| data.projectAdmin[].userName | String | 管理员姓名 |
| data.projectAdmin[].userEmail | String | 管理员邮箱 |

---

## 20. op=User.getToken — 签发新令牌

### 请求格式
{"op": "User.getToken", "action": "user", "data": {"expireTime": ..., "projectid": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| expireTime | Long | 否 | 新令牌过期时间（`getLong`） |
| projectid | Integer | 否 | 目标项目 ID（缺省从在线信息取） |

### 核心逻辑

从 `apirequest.getOnlineUserInfo()` 取出当前用户信息，用库中加密密码重新调用 `userService.login()` 签发新 sid。需当前会话已鉴权。

### 响应

`{ error_code, msg, data: { result: sid字符串 } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | String | 新登录令牌（sid） |

---

### 涉及表

- `db_user`（`DbUserInfo`）
- `db_user_role`（`DbUserRoleInfo`）
- `db_user_project_role`（`DbUserProjectRoleInfo`）
- `view_enterprise_user`（`ViewEnterpriseUser`）
- `view_user_enterprise_project_role`（`ViewUserEnterpriseProjectRole`）
