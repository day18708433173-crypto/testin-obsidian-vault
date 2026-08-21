# ProjectGlobalVariablesController — 项目全局变量控制器

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/project/ProjectGlobalVariablesController.java`
> 类级路由：`/datasource/project/global_variables`（完整前缀 `/openapi/v3/datasource/project/global_variables`）
> Service 实现：`ProjectGlobalVariablesServiceImpl`
> 业务：项目级别全局变量的 CRUD + RSA 加密 + 密钥获取。支持普通变量和加密变量。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 1 | GET | `/v3/datasource/project/global_variables/` | list | 分页查询全局变量列表 |
| 2 | POST | `/v3/datasource/project/global_variables/` | add | 新增全局变量 |
| 3 | PUT | `/v3/datasource/project/global_variables/` | edit | 按 ID 编辑全局变量 |
| 4 | PUT | `/v3/datasource/project/global_variables/by_var_name` | editByVarName | 按变量名编辑 |
| 5 | DELETE | `/v3/datasource/project/global_variables/` | del | 删除全局变量 |
| 6 | POST | `/v3/datasource/project/global_variables/encryption` | encryptionVar | RSA 加密变量值 |
| 7 | POST | `/v3/datasource/project/global_variables/getSecretKey` | getSecretKey | 获取 RSA 公钥 |

统一响应包装：`ResponseResult<T>`。

涉及表：`project_global_variables`。

---

## 1. GET / — 分页查询全局变量

### 入口

`ProjectGlobalVariablesController.list(projectId, page=1, pageSize=10, type?, variableName?)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| project_id | Integer | 是 | — | 项目 ID |
| page | Integer | 否 | 1 | 页码 |
| page_size | Integer | 否 | 10 | 每页条数 |
| type | Integer | 否 | — | 变量类型：1=普通 2=加密 |
| variable_name | String | 否 | — | 变量名模糊筛选 |

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.totalRow | Long | 总记录数 |
| data.totalPage | Long | 总页数 |
| data.pageSize | Integer | 每页条数 |
| data.page | Integer | 页码 |
| data.list | List\<ProjectGlobalVariablesDTO\> | 全局变量列表 |
| data.list[].id | Integer | 主键 |
| data.list[].projectId | Integer | 项目 id |
| data.list[].type | Integer | 1 普通变量 2 加密变量 |
| data.list[].variableName | String | 变量名 |
| data.list[].variableValue | String | 变量值（加密变量存密文） |
| data.list[].variableType | Integer | 数据类型 0 默认 1 String 2 数字 3 对象 4 数组 |
| data.list[].variableDesc | String | 变量描述 |
| data.list[].varSecret | Integer | 是否加密类型变量 1 是 0 否 |
| data.list[].status | Integer | 0 逻辑删除 1 正常 |
| data.list[].createtime / updatetime | Long | 创建/更新时间 |

`ResponseResult<PageInfoList<ProjectGlobalVariablesDTO>>`

---

## 2. POST / — 新增全局变量

### 入口

`ProjectGlobalVariablesController.add(@RequestBody ProjectGlobalVariablesDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 否 | 项目 ID |
| type | Integer | 否 | 1=普通变量 2=加密变量 |
| variableName | String | 否 | 变量名 |
| variableValue | String | 否 | 变量值（加密变量存加密后的值） |
| variableType | Integer | 否 | 数据类型：0=默认 1=String 2=数字 3=对象 4=数组 |
| variableDesc | String | 否 | 变量描述 |
| userId / userName | — | 否 | 操作人 |

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data | Integer | 新增记录的 ID（或影响行数） |

`ResponseResult<Integer>`，data = 新增记录的 ID。

---

## 3-4. 编辑全局变量

| 接口 | 路径 | 说明 |
|------|------|------|
| PUT `/` | edit | 按 ID 更新 |
| PUT `/by_var_name` | editByVarName | 按变量名 + projectId 更新（适合只知变量名的场景） |

### 3. PUT / — 按 ID 编辑

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Integer | 否 | 主键 |
| projectId / type / variableName / variableValue / variableType / variableDesc | — | 否 | 同新增 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data | Integer | 影响行数 |

### 4. PUT /by_var_name — 按变量名编辑

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 否 | 项目 id |
| variableName | String | 否 | 变量名 |
| variableValue / variableType / variableDesc | — | 否 | 其它字段 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data | Integer | 影响行数 |

---

## 5. DELETE / — 删除全局变量

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| project_id | Integer | 是 | 项目 ID |
| id | Integer | 是 | 变量记录 ID |
| user_id | Integer | 否 | 操作人 ID |
| user_name | String | 否 | 操作人名称 |

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data | Integer | 影响行数 |

`ResponseResult<Integer>`，MyBatis Plus 逻辑删除（status→0）。

---

## 6. POST /encryption — RSA 加密变量值

### 入口

`ProjectGlobalVariablesController.encryptionVar(@RequestBody ProjectGlobalVariablesDTO)`

### 请求参数（JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| variableValue | String | 否 | 待加密变量值 |
| （其它 ProjectGlobalVariablesDTO 字段） | — | 否 | — |

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data | String | 加密后的字符串 |

### 实现意图

前端将明文变量值发送到此接口，服务端使用 RSA 公钥加密后返回密文，前端再保存密文。

---

## 7. POST /getSecretKey — 获取 RSA 公钥

### 入口

`ProjectGlobalVariablesController.getSecretKey()`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| — | — | — | 无业务参数 |

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.result | Object | RSA 公钥（base64） |

`ResponseResult<Map<String, Object>>`，`data.result` = RSA 公钥（base64）。

### 实现意图

供上位机（设备端）获取 RSA 公钥，用于解密项目全局加密变量。
