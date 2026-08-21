# ProjectController — 项目组控制器

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/ProjectController.java`
> 类级路由：`/project`
> Service 实现：`cn.testin.business.impl.user.ProjectService`、`cn.testin.business.impl.user.UserService`
> 业务：项目组（Project）信息的增删改查、项目组成员管理、用户列表查询。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|---|---|---|---|
| 1 | PUT | `/v3/project/{project_id}` | updateProjectInfo | 更新项目组信息 |
| 2 | GET | `/v3/project/users` | getProjectUsers | 查询项目组成员列表（分页，支持按类型查询项目内/外） |
| 3 | GET | `/v3/project/{project_id}` | getProjectInfo | 查询单个项目组详情 |
| 4 | POST | `/v3/project/operate_users` | operateUser | 操作项目组成员（添加/移除/修改角色） |
| 5 | GET | `/v3/project/list` | getProjectList | 查询企业下项目组列表 |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`。
GET `/v3/project/users` 使用 `@UnderlineToCamel`，下划线 query 参数自动转驼峰。

涉及表：`db_project`（`DbProjectInfo`）、`db_enterprise_user_relation`（`DbEnterpriseUserRelation`）、`view_enterprise_user`（`ViewEnterpriseUser`）。

---

## 1. PUT /v3/project/{project_id} — 更新项目组信息

### 入口

`ProjectController.updateProjectInfo(@RequestBody @Validate ProjectUpdateRequestDTO dto, @PathVariable("project_id") int projectId)`

### 请求参数

**Path**：

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| project_id | Integer | 是 | 项目组 ID（路径变量，代码中 setProjectId 回填 DTO） |

**Body**（ProjectUpdateRequestDTO）：

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectName | String | 否 | 项目名称 |
| projectDescription | String | 否 | 项目描述 |
| productNo | String | 否 | 产品技术编号 |
| thirdPartyProjectId | String | 否 | 第三方项目 ID |
| ciccSessionId | String | 否 | 中金公司内部系统 sessionId |
| eid | Integer | 否 | 企业 ID（继承 BaseRequestDTO） |
| projectId | Integer | 否 | 项目 ID（继承 BaseRequestDTO，会被路径覆盖） |
| userId | Integer | 否 | 用户 ID（继承 BaseRequestDTO） |
| userName | String | 否 | 用户名（继承 BaseRequestDTO） |

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

根据路径中的 projectId 更新项目组信息。

### 调用链

```
ProjectController.updateProjectInfo
└─ ProjectService.modifyProjectInfo
```

---

## 2. GET /v3/project/users — 查询项目组成员

### 入口

`ProjectController.getProjectUsers(@UnderlineToCamel ProjectUserRequestDTO request)`

### 请求参数（Query，下划线转驼峰）

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| page | Integer | 否 | `PAGE_DEFAULT` | 页码 |
| page_size | Integer | 否 | `PAGE_SIZE_DEFAULT` | 每页条数 |
| type | String | 否 | `IN_PROJECT` | 查询类型：`IN_PROJECT` 查项目内成员，其他值查不在项目中的用户 |
| project_id | Integer | 是 | — | 项目 ID（`@NotNull`，继承 BaseQueryRequestDTO） |
| user_id | Integer | 是 | — | 用户 ID（`@NotNull`，继承 BaseQueryRequestDTO） |
| eid | Integer | 否 | — | 企业 ID（继承 BaseQueryRequestDTO） |
| keywords | String | 否 | — | 搜索关键字 |
| name | String | 否 | — | 姓名筛选 |
| email | String | 否 | — | 邮箱筛选 |
| role_id | Integer | 否 | — | 角色 ID 筛选 |

### 响应结构

`ResponseResult<BaseList<UserInfo>>`，分页列表。

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象（BaseList） |
| data.list | Array | 用户列表（UserInfo） |
| data.list[].id | Integer | 用户 ID |
| data.list[].userName | String | 用户名称 |
| data.list[].userEmail | String | 用户邮箱 |
| data.list[].userMobile | String | 用户手机号 |
| data.list[].userJob | String | 用户职位 |
| data.list[].roleId | Integer | 权限 ID |
| data.list[].roleName | String | 权限名称 |
| data.page / pageSize / totalRow / totalPage | Integer | 分页信息 |

### 实现意图

根据 type 参数区分：查询项目组内的成员列表（走 `userService.getProjectUserList`），或查询不在该项目组中的用户列表（走 `projectService.getUsersNotInProject`）。最后统一转换为 `UserInfo`。

### 关联横切

- `@UnderlineToCamel`：自动将下划线 query 参数转驼峰。

---

## 3. GET /v3/project/{project_id} — 查询项目组详情

### 入口

`ProjectController.getProjectInfo(@PathVariable("project_id") int projectId)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| project_id | Integer | 是 | 项目组 ID（路径变量） |

### 响应结构

`ResponseResult<ProjectInfoVo>`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 项目详情对象（ProjectInfoVo） |
| data.projectId | Integer | 项目 ID |
| data.projectName | String | 项目名称 |
| data.thirdPartyProjectId | String | 第三方项目 ID |
| data.projectDescription | String | 项目描述 |
| data.productNo | String | 产品技术编号 |
| data.extend | String | 扩展字段 |

### 实现意图

根据项目组 ID 查询项目详情，将 `DbProjectInfo` 转为前端友好的 `ProjectInfoVo`（builder 模式）。

### 调用链

```
ProjectController.getProjectInfo
└─ ProjectService.get(projectId)
```

---

## 4. POST /v3/project/operate_users — 操作项目组成员

### 入口

`ProjectController.operateUser(@RequestBody @Validate OperateUserRequestDTO operateUserRequestDTO)`

### 请求参数（JSON Body，OperateUserRequestDTO）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| operate | String | 是 | 操作类型（`@NotNull`）：1=添加，2=删除 |
| targetUserIdList | List\<Integer\> | 是 | 待操作的用户 ID 列表（代码中 `CollectionUtils.isEmpty` 校验不能为空） |
| targetUserId | Integer | 否 | 单个待操作的用户 ID |
| roleId | Integer | 否 | 角色 ID |
| targetProjectId | Integer | 否 | 应在哪个项目组里被操作 |
| newUserInfo | Object(UserInfo) | 否 | 需要被添加的用户信息 |
| updateUserInfo | Object(UpdateUserInfoDTO) | 否 | 需要被修改的用户角色 |
| eid | Integer | 否 | 企业 ID（继承 BaseRequestDTO） |
| projectId | Integer | 否 | 项目 ID（继承 BaseRequestDTO） |
| userId | Integer | 否 | 用户 ID（继承 BaseRequestDTO） |
| userName | String | 否 | 用户名（继承 BaseRequestDTO） |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 操作影响记录数。

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Long | 操作影响记录数 |

### 实现意图

批量操作项目组成员：根据 `op` 值执行添加、移除或角色变更。

### 调用链

```
ProjectController.operateUser
└─ ProjectService.operateUser
```

---

## 5. GET /v3/project/list — 查询项目组列表

### 入口

`ProjectController.getProjectList(@RequestParam(value = "eid", required = false) Integer eid)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| eid | Integer | 否 | 1 | 企业 ID（不传或 <=0 默认为 1） |

### 响应结构

`ResponseResult<List<ProjectInfoVo>>`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Array | 项目组列表（ProjectInfoVo） |
| data[].projectId | Integer | 项目 ID |
| data[].projectName | String | 项目名称 |
| data[].thirdPartyProjectId | String | 第三方项目 ID |
| data[].projectDescription | String | 项目描述 |
| data[].productNo | String | 产品技术编号 |
| data[].extend | String | 扩展字段 |

### 实现意图

查询指定企业下的全部项目组列表，以 `ProjectInfoVo` 返回。

### 调用链

```
ProjectController.getProjectList
└─ ProjectService.getProjectListByEid(eid)
```
