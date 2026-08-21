# EnterpriseController — 企业控制器

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/EnterpriseController.java`
> 类级路由：`/v3/user/enterprise`
> Service 实现：`cn.testin.business.impl.user.EnterpriseServiceImpl`
> 业务：企业成员管理、批量导入用户。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|---|---|---|---|
| 1 | POST | `/v3/user/enterprise/operate_users` | operateEnterpriseUsers | 操作企业成员（添加/移除/修改角色等） |
| 2 | POST | `/v3/user/enterprise/import_users` | importUserInfoList | 批量导入用户到企业 |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`。

涉及表：`db_enterprise_user_relation`（`DbEnterpriseUserRelation`）、`db_user`（`DbUserInfo`），底层通过 `EnterpriseServiceImpl` 操作。

---

## 1. POST /v3/user/enterprise/operate_users — 操作企业成员

### 入口

`EnterpriseController.operateEnterpriseUsers(@RequestBody @Validate OperateUserRequestDTO operateUserRequestDTO)`

### 请求参数（JSON Body，OperateUserRequestDTO）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| operate | String | 是 | 操作类型（`@NotNull`）：1=添加，2=删除 |
| targetUserIdList | List\<Integer\> | 否 | 待操作的用户 ID 列表 |
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

`ResponseResult<BaseDataResultDTO>`，`data.result` = 操作影响记录数（Long）。

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Long | 操作影响记录数 |

### 实现意图

对企业下的成员执行批量操作（添加成员、移除成员、修改成员角色等），由 `op` 参数控制操作类型。

### 调用链

```
EnterpriseController.operateEnterpriseUsers
└─ EnterpriseServiceImpl.operateEnterpriseUsers
```

---

## 2. POST /v3/user/enterprise/import_users — 批量导入用户

### 入口

`EnterpriseController.importUserInfoList(@RequestBody @Validate ImportUserInfoListDTO importUserInfoListDTO)`

### 请求参数（JSON Body，ImportUserInfoListDTO）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| importFileUrl | String | 是 | 导入文件的 URL（`@NotNull`） |
| type | Integer | 是 | 导入类型（`@NotNull`）：0=不覆盖原有用户，1=覆盖原有用户 |
| projectIdList | List\<Integer\> | 是 | 项目 ID 列表（`@NotNull`） |
| eid | Integer | 否 | 企业 ID（继承 BaseRequestDTO） |
| projectId | Integer | 否 | 项目 ID（继承 BaseRequestDTO） |
| userId | Integer | 否 | 用户 ID（继承 BaseRequestDTO） |
| userName | String | 否 | 用户名（继承 BaseRequestDTO） |

### 响应结构

`ResponseResult<List<ImportUserInfoResultDTO>>`，返回每个用户的导入结果（成功/失败及原因）。

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Array | 每个用户的导入结果列表（ImportUserInfoResultDTO） |
| data[].status | Integer | 是否导入成功：0=成功，1=失败 |
| data[].id | Integer | 用户 ID |
| data[].userName | String | 用户名称 |
| data[].userEmail | String | 用户邮箱 |
| data[].userMobile | String | 用户手机号 |
| data[].userJob | String | 用户职位 |
| data[].roleId | Integer | 权限 ID |
| data[].roleName | String | 权限名称 |

### 实现意图

批量将用户导入到指定企业中，返回每个用户的导入状态及错误信息。

### 调用链

```
EnterpriseController.importUserInfoList
└─ EnterpriseServiceImpl.importUserList
```
