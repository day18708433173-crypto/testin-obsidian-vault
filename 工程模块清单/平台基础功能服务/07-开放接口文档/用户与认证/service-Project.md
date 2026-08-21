# service-Project — 项目组接口（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/user/Project.java`
> 基础类：`cn.testin.service.GenericBaseService`
> 机制：请求走 `ApiServlet /*` 入口，通过 `action=user` + `op=Project.方法名` 路由到此类的对应 public 方法；每个方法的参数为 `ApiRequest`，返回 JSON 字符串。
> - **action**: `user`（对应包 `cn.testin.service.user`）
> - **入口格式**：`{"op": "Project.方法名", "action": "user", "data": {...}}`
> 业务：项目组 CRUD、项目组成员管理、访问时间更新、分配/定制化接口等。

## op 列表总表

| # | op | 方法名 | 说明 |
|---|---|---|---|
| 1 | Project.getProjectList | getProjectList | 查询项目组列表（分页，支持条件筛选） |
| 2 | Project.getUserProjectList | getUserProjectList | 查询某用户下的项目组列表 |
| 3 | Project.addProject | addProject | 新增项目组 |
| 4 | Project.modifyProject | modifyProject | 更新项目组信息（支持第三方ID定位） |
| 5 | Project.modifyAccessTime | modifyAccessTime | 更新项目组访问时间（切项目触发） |
| 6 | Project.getDefaultProject | getDefaultProject | 查询用户的默认项目组 |
| 7 | Project.get | get | 根据项目组 ID 获取项目详情 |
| 8 | Project.operateUser | operateUser | 操作项目组下的用户（添加/移除） |
| 9 | Project.assign | assign | 项目组分配（用户不存在则创建，项目不存在则创建） |
| 10 | Project.hengsunCustomize | hengsunCustomize | 恒生定制化接口（自动创建用户+项目+关联） |
| 11 | Project.getByThirdPartyProjectid | getByThirdPartyProjectid | 根据第三方项目 ID 获取项目信息 |
| 12 | Project.addUser | addUser | 新增成员到项目组并设置角色 |
| 13 | Project.removeUser | removeUser | 从项目组移除成员 |
| 14 | Project.maintainRole | maintainRole | 修改项目组成员角色 |

---

## 返回结构约定

所有方法返回 V1 信封：`{ error_code, msg, data }`。`data` 内按方法不同含以下键之一：

| 键 | 类型 | 说明 |
|---|---|---|
| result | Integer/Boolean/String | 结果值（影响行数、主键、布尔结果等），具体见各方法「响应」 |
| object | Object | 单个对象 |
| list | Array | 列表 |
| page / pageSize / totalRow / totalPage | Integer | 分页信息（list 伴随） |

各方法 `object`/`list` 内部字段见对应「响应」小节。

---

## 1. op=Project.getProjectList — 项目组列表

### 请求格式
{"op": "Project.getProjectList", "action": "user", "data": {"eid": ..., "conditions": {...}, "page": ..., "pageSize": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID（缺省从在线信息取） |
| conditions | JSONObject | 否 | 筛选条件对象：`{ keywords, productNo, status, projectIds }` |
| page | Integer | 否 | 页码 |
| pageSize | Integer | 否 | 每页条数 |
| status | String | 否 | 状态（直接在根级也可传） |

### 核心逻辑

多条件分页查询 `db_project` 表，结果转为 JSON 列表，含 `projectid`、`projectName`、`projectDescr`、`projectProductNo`、`projectStatus`、`eid`、`projectCreatetime`、`thirdPartyProjectid`、`extend`。

### 响应

`{ error_code, msg, data: { list: [...], page, pageSize, totalRow, totalPage } }`

---

## 2. op=Project.getUserProjectList — 用户的项目组列表

### 请求格式
{"op": "Project.getUserProjectList", "action": "user", "data": {"eid": ..., "userid": ..., "page": ..., "pageSize": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID（缺省从在线信息取） |
| userid | Integer | 否 | 用户 ID（缺省从在线信息取） |
| checkValid | Boolean | 否 | 合法性校验标志 |
| page / pageSize | Integer | 否 | 分页 |
| conditions.keywords | String | 否 | 搜索关键字 |
| userInfo.userId | Integer | 否 | OpenAPI 风格传 userId |

### 核心逻辑

查询某用户关联的项目组（含角色、访问时间等），从 `view_enterprise_user` 视图取数据。

### 响应

`{ error_code, msg, data: { list: [...], page, pageSize, totalRow, totalPage } }`

---

## 3. op=Project.addProject — 新增项目组

### 请求格式
{"op": "Project.addProject", "action": "user", "data": {"eid": ..., "projectName": "...", ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID（缺省从在线信息取） |
| projectName | String | 是 | 项目组名称 |
| projectDescr / projectDescription | String | 否 | 项目描述 |
| projectProductNo | String | 否 | 产品技术编号 |
| thirdPartyProjectid | String | 否 | 第三方项目 ID |

### 核心逻辑

构造 `DbProjectInfo`，调用 `projectService.addProject()` 写入 `db_project`。

### 响应

`{ error_code, msg, data: { result: projectId } }`

---

## 4. op=Project.modifyProject — 更新项目组

### 请求格式
{"op": "Project.modifyProject", "action": "user", "data": {"projectid": ..., "projectName": "...", ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID |
| projectid | Integer | 是（或 thirdPartyProjectid） | 项目 ID |
| projectName | String | 否 | 新名称 |
| projectDescr | String | 否 | 新描述 |
| projectProductNo | String | 否 | 新产品编号 |
| thirdPartyProjectid | String | 否 | 第三方 ID（可用于定位项目） |
| extend | String | 否 | 扩展信息 JSON 字符串 |
| status | String | 否 | 状态（必须为 ON 或 OFF） |

### 核心逻辑

如果传了 `thirdPartyProjectid` 但未传 `projectid`，则先通过第三方 ID 查出项目 ID 再更新。支持部分字段更新。

### 响应

`{ error_code, msg, data: { result } }`

---

## 5. op=Project.modifyAccessTime — 更新访问时间

### 请求格式
{"op": "Project.modifyAccessTime", "action": "user", "data": {"eid": ..., "projectid": ..., "userid": ..., "sid": "..."}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID |
| projectid | Integer | 否 | 项目组 ID |
| userid | Integer | 否 | 用户 ID |
| sid | String | 否 | 令牌（用于同步更新 `db_online.projectid`） |

### 核心逻辑

更新 `db_enterprise_user_relation` 中的 `p_access_time`，并更新 `db_online` 中的 `projectid`，同时发送缓存清除通知 `NoticeUtil.sendNotice("delete", "EnterpriseUserRelation", ...)`。

### 响应

`{ error_code, msg, data: { result } }`

---

## 6. op=Project.getDefaultProject — 默认项目组

### 请求格式
{"op": "Project.getDefaultProject", "action": "user", "data": {"eid": ..., "userid": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID |
| userid | Integer | 否 | 用户 ID |

### 核心逻辑

查询用户上次访问的项目组（默认项目）。

### 响应

`{ error_code, msg, data: { object: { projectid, projectName, projectDescr, projectStatus, projectRoleid, projectAccesstime } } }`

---

## 7. op=Project.get — 项目详情

### 请求格式
{"op": "Project.get", "action": "user", "data": {"projectid": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectid | Integer | 是 | 项目 ID |

### 核心逻辑

调用 `projectService.get(projectid)` 查 `db_project`，不存在时返回 `noneData`。

### 响应

`{ error_code, msg, data: { object: { projectid, projectName, projectDescr, projectProductNo, projectStatus, eid, projectCreatetime, thirdPartyProjectid, extend } } }`

---

## 8. op=Project.operateUser — 操作项目组成员

### 请求格式
{"op": "Project.operateUser", "action": "user", "data": {"eid": ..., "projectid": ..., "userid": ..., "op": ..., "roleid": ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID |
| projectid | Integer | 是 | 项目 ID |
| userid | Integer | 是 | 用户 ID |
| op | String | 是 | 操作类型 |
| roleid | Integer | 否 | 角色 ID |
| userInfo | JSONObject | 否 | OpenAPI 风格（userInfo.userId / userInfo.projectId） |

### 核心逻辑

调用 `projectService.operateUser(eid, projectid, userid, op, roleid)`，成功后发送缓存清除通知。

### 响应

`{ error_code, msg, data: { result } }`

---

## 9. op=Project.assign — 项目组分配（自动创建）

### 请求格式
{"op": "Project.assign", "action": "user", "data": {"username": "...", "productNo": "...", "projectName": "...", ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| username | String | 是 | 用户名/邮箱 |
| productNo | String | 是 | 产品技术编号 |
| projectName | String | 是 | 项目组名称 |
| projectId | Integer | 否 | 已有项目 ID（传入则复用并更新 info） |

### 核心逻辑

若用户不存在则自动创建用户，若项目不存在则自动创建项目，若关联关系不存在则创建关联关系。保障用户、项目、关联三者俱备。

### 响应

`{ error_code, msg, data: { result: 1, projectId } }`

---

## 10. op=Project.hengsunCustomize — 恒生定制接口

### 请求格式
{"op": "Project.hengsunCustomize", "action": "user", "data": {"vId": "...", "username": "...", "hundsunProjectId": "...", ...}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| vId | String | 是 | 版本 ID |
| username | String | 是 | 用户邮箱 |
| hundsunProjectId | String | 是 | 恒生项目 ID |
| hundsunProjectName | String | 否 | 项目名称 |
| realname | String | 否 | 用户真实姓名 |

### 核心逻辑

类似 `assign` 但参数来源为恒生系统格式。判断项目是否存在（用 `vId_hundsunProjectId` 拼接为 productNo），不存在则创建；用户不存在则创建；关联关系不存在则创建；若用户不在线则执行登录；更新在线表中的 projectid。

### 响应

`{ error_code, msg, data: { object: { userid, projectid, eid, sid } } }`

---

## 11. op=Project.getByThirdPartyProjectid — 按第三方 ID 查项目

### 请求格式
{"op": "Project.getByThirdPartyProjectid", "action": "user", "data": {"thirdPartyProjectid": "..."}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| thirdPartyProjectid | String | 是 | 第三方项目 ID |

### 核心逻辑

调用 `projectService.getByThirdProjectId()` 查询。

### 响应

同 `get` 方法。

---

## 12. op=Project.addUser — 新增成员到项目组

### 请求格式
{"op": "Project.addUser", "action": "user", "data": {"eid": ..., "userInfo": {"userid": ..., "projectid": ..., "roleid": ...}}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID |
| userInfo.thirdPartyUserid | String | 二选一 | 第三方用户 ID |
| userInfo.userid | Integer | 二选一 | 用户 ID |
| userInfo.projectid | Integer | 是 | 项目 ID |
| userInfo.roleid | Integer | 是 | 角色 ID |
| userInfo.thirdPartyProjectid | String | 否 | 第三方项目 ID（与 thirdPartyUserid 配合使用） |

### 核心逻辑

验证用户存在且属于当前企业后，调用 `projectService.addUser(eid, projectid, user, roleid)` 创建项目-用户关联。

### 响应

`{ error_code, msg, data: { result } }`

---

## 13. op=Project.removeUser — 移除项目组成员

### 请求格式
{"op": "Project.removeUser", "action": "user", "data": {"eid": ..., "userInfo": {"userid": ..., "projectid": ...}}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID |
| userInfo.thirdPartyUserid | String | 二选一 | 第三方用户 ID |
| userInfo.userid | Integer | 二选一 | 用户 ID |
| userInfo.projectid | Integer | 是 | 项目 ID |
| userInfo.thirdPartyProjectid | String | 否 | 第三方项目 ID |

### 核心逻辑

验证用户存在后，调用 `projectService.removeUser(eid, projectid, userId)` 删除关联。

### 响应

`{ error_code, msg, data: { result } }`

---

## 14. op=Project.maintainRole — 修改项目组成员角色

### 请求格式
{"op": "Project.maintainRole", "action": "user", "data": {"eid": ..., "userInfo": {"userid": ..., "projectid": ..., "roleid": ...}}}

### 请求参数

| JSON 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID |
| userInfo.thirdPartyUserid | String | 二选一 | 第三方用户 ID |
| userInfo.userid | Integer | 二选一 | 用户 ID |
| userInfo.projectid | Integer | 是 | 项目 ID |
| userInfo.roleid | Integer | 是 | 新角色 ID |
| userInfo.thirdPartyProjectid | String | 否 | 第三方项目 ID |

### 核心逻辑

验证用户存在后，调用 `projectService.maintainRole(eid, projectid, user, roleid)` 更新角色。

### 响应

`{ error_code, msg, data: { result } }`

---

### 涉及表

- `db_project`（`DbProjectInfo`）— 项目组信息
- `db_enterprise_user_relation`（`DbEnterpriseUserRelation`）— 企业用户项目关联
- `db_online`（`DbUserOnline`）— 在线状态表
- `db_user`（`DbUserInfo`）— 用户表（用于 assign/hengsunCustomize 中的自动化创建）
