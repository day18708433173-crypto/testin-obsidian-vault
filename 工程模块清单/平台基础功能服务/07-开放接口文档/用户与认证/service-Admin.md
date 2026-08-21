# service-Admin — 管理后台角色接口（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/user/Admin.java`
> 基础类：`cn.testin.service.GenericBaseService`
> 机制：请求走 `ApiServlet /*` 入口，通过 `action=user` + `op=Admin.方法名` 路由到此类的对应 public 方法；每个方法的参数为 `ApiRequest`，返回 JSON 字符串。
> - **action**: `user`（对应包 `cn.testin.service.user`）
> - **入口格式**：`{"op": "Admin.方法名", "action": "user", "data": {...}}`
> 业务：管理后台用户角色查询。

## op 列表总表

| # | op | 方法名 | 说明 |
|---|---|---|---|
| 1 | Admin.role | role | 根据 userid 查询管理后台角色 |

---

## 1. op=Admin.role — 查询管理后台角色

### 请求格式
{"op": "Admin.role", "action": "user", "data": {"userid": ...}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| userid | Integer | 是 | 用户 ID（optInt 读取后判 0，等于 0 时返回参数错误） |

### 核心逻辑

查 `db_admin_user_role`（`DbAdminUserRole`），根据 userid 获取用户在管理后台的角色信息。用户 ID 无效时返回参数错误。

### 响应

`{ error_code, msg, data: { object: DbAdminUserRole } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 管理后台用户角色对象 |
| data.object.userId | Integer | 用户 ID |
| data.object.roleId | Integer | 角色 ID |
| data.object.status | Integer | 状态 |
| data.object.createTime | Long | 创建时间 |
| data.object.updateTime | Long | 更新时间 |

### 涉及表

- `db_admin_user_role`（`DbAdminUserRole`）— `IAdminUserRoleDAO`
