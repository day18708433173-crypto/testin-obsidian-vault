# service-Role — 角色接口（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/user/Role.java`
> 基础类：`cn.testin.service.GenericBaseService`
> 机制：请求走 `ApiServlet /*` 入口，通过 `action=user` + `op=Role.方法名` 路由到此类的对应 public 方法；每个方法的参数为 `ApiRequest`，返回 JSON 字符串。
> - **action**: `user`（对应包 `cn.testin.service.user`）
> - **入口格式**：`{"op": "Role.方法名", "action": "user", "data": {...}}`
> 业务：角色 CRUD、按角色查用户、项目内角色判定等。

## op 列表总表

| # | op | 方法名 | 说明 |
|---|---|---|---|
| 1 | Role.addRole | addRole | 新增角色 |
| 2 | Role.modifyRole | modifyRole | 修改角色信息 |
| 3 | Role.getRoleList | getRoleList | 查询角色列表（可选过滤管理员） |
| 4 | Role.getRoleById | getRoleById | 按 roleId 查角色 |
| 5 | Role.getRoleByName | getRoleByName | 按 roleName 查角色 |
| 6 | Role.getUserListByRoleId | getUserListByRoleId | 根据角色查用户列表 |
| 7 | Role.deleteRole | deleteRole | 删除角色 |
| 8 | Role.roleByEidAndProjectId | roleByEidAndProjectId | 根据 eid + projectId + uid 获取用户的项目内角色 |
| 9 | Role.roleByUid | roleByUid | 根据 uid 获取用户在各项目中的角色列表 |

---

## 1. op=Role.addRole — 新增角色

### 请求格式
{"op": "Role.addRole", "action": "user", "data": {"name": "...", "descr": "...", "eid": ...}}

### 请求参数

JSON 根级或 `roleInfo` 子对象，解析为 `DbRoleInfo`；`name` 为空时返回参数错误。

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| name | String | 是 | 角色名称（`StringUtils.isEmpty` 判空） |
| descr | String | 否 | 角色描述 |
| eid | Integer | 否 | 企业 ID |
| status | Integer | 否 | 角色状态 |

### 核心逻辑

调用 `iroleservice.addRole()` 写入 `db_role`。

### 响应

`{ error_code, msg, data: { result: roleId } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 新增角色 ID（roleId） |

---

## 2. op=Role.modifyRole — 修改角色

### 请求格式
{"op": "Role.modifyRole", "action": "user", "data": {"id": ..., "name": "...", ...}}

### 请求参数

`DbRoleInfo` JSON，`id` 必填（`id < 1` 返回参数错误）。

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Integer | 是 | 角色 ID（须 >= 1） |
| name | String | 否 | 角色名称 |
| descr | String | 否 | 角色描述 |
| status | Integer | 否 | 角色状态 |
| eid | Integer | 否 | 企业 ID |

### 核心逻辑

调用 `iroleservice.modifyRoleInfo()`。

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

## 3. op=Role.getRoleList — 角色列表

### 请求格式
{"op": "Role.getRoleList", "action": "user", "data": {"eid": ..., "excludeAdminRole": 0|1}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业 ID（null 时返回参数错误） |
| excludeAdminRole | Integer | 否 | 1=过滤掉系统管理员角色（0 不过滤） |

### 核心逻辑

查询企业下全部角色，`excludeAdminRole=1` 时过滤 `SYSTEM_ADMIN`。

### 响应

`{ error_code, msg, data: { list: [ { roleId, roleName, roleDescr }, ... ] } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 角色列表 |
| data.list[].roleId | Integer | 角色 ID |
| data.list[].roleName | String | 角色名称 |
| data.list[].roleDescr | String | 角色描述 |

---

## 4. op=Role.getRoleById — 按 ID 查角色

### 请求格式
{"op": "Role.getRoleById", "action": "user", "data": {"roleId": ..., "eid": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| roleId | Integer | 是 | 角色 ID |
| eid | Integer | 否 | 企业 ID |

### 响应

`{ error_code, msg, data: { object: { roleId, roleName, roleDescr } } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 角色对象（不存在时为空对象） |
| data.object.roleId | Integer | 角色 ID |
| data.object.roleName | String | 角色名称 |
| data.object.roleDescr | String | 角色描述 |

---

## 5. op=Role.getRoleByName — 按名称查角色

### 请求格式
{"op": "Role.getRoleByName", "action": "user", "data": {"roleName": "...", "eid": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| roleName | String | 是 | 角色名称 |
| eid | Integer | 否 | 企业 ID |

### 响应

同 `getRoleById`。

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 角色对象（不存在时为空对象） |
| data.object.roleId | Integer | 角色 ID |
| data.object.roleName | String | 角色名称 |
| data.object.roleDescr | String | 角色描述 |

---

## 6. op=Role.getUserListByRoleId — 按角色查用户

### 请求格式
{"op": "Role.getUserListByRoleId", "action": "user", "data": {"roleId": ..., "eid": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| roleId | Integer | 是 | 角色 ID |
| eid | Integer | 否 | 企业 ID |

### 核心逻辑

查询拥有指定角色的全部用户，返回用户基本信息列表。

### 响应

`{ error_code, msg, data: { list: [ { userid, email, name, mobile, job, status, createTime }, ... ] } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 用户列表 |
| data.list[].userid | Integer | 用户 ID |
| data.list[].email | String | 用户邮箱 |
| data.list[].name | String | 用户姓名 |
| data.list[].mobile | String | 用户手机 |
| data.list[].job | String | 用户职位 |
| data.list[].status | Integer | 用户状态 |
| data.list[].createTime | Long | 创建时间 |

---

## 7. op=Role.deleteRole — 删除角色

### 请求格式
{"op": "Role.deleteRole", "action": "user", "data": {"roleId": ..., "eid": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| roleId | Integer | 是 | 角色 ID |
| eid | Integer | 是 | 企业 ID |

### 响应

`{ error_code, msg, data: { result: roleId } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 删除的角色 ID |

---

## 8. op=Role.roleByEidAndProjectId — 项目内角色

### 请求格式
{"op": "Role.roleByEidAndProjectId", "action": "user", "data": {"eid": ..., "projectId": ..., "uid": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 是 | 企业 ID（null 返回参数错误） |
| projectId | Integer | 是 | 项目 ID（null 返回参数错误） |
| uid | Integer | 否 | 用户 ID |

### 核心逻辑

先查用户默认角色，若为 SYSTEM_ADMIN 或 ENTERPRISE_ADMIN 则直接返回该角色；否则查 `db_enterprise_user_relation` 中该用户在该项目下的角色。

### 响应

`{ error_code, msg, data: { result: roleId } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 项目内角色 ID |

---

## 9. op=Role.roleByUid — 用户各项目角色

### 请求格式
{"op": "Role.roleByUid", "action": "user", "data": {"uid": ..., "eid": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| uid | Integer | 是 | 用户 ID（null 返回参数错误） |
| eid | Integer | 是 | 企业 ID（null 返回参数错误） |

### 核心逻辑

查询用户在各项目中的不同角色（去重），过滤掉 ENTERPRISE_ADMIN 和 SYSTEM_ADMIN。

### 响应

`{ error_code, msg, data: { list: [ { roleId, roleName, roleDescr }, ... ] } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 用户在各项目中的角色列表 |
| data.list[].roleId | Integer | 角色 ID |
| data.list[].roleName | String | 角色名称 |
| data.list[].roleDescr | String | 角色描述 |

---

### 涉及表

- `db_role`（`DbRoleInfo`）— 角色表
- `db_user`（`DbUserInfo`）— 用户表
- `db_user_role`（`DbUserRoleInfo`）— 用户默认角色
- `db_enterprise_user_relation`（`DbEnterpriseUserRelation`）— 企业用户项目关联
