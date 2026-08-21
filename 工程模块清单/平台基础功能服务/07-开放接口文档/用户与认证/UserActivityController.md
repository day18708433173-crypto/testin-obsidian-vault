# UserActivityController — 用户活跃度控制器

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/UserActivityController.java`
> 类级路由：`/v3/user/user_activity`
> Service 实现：`cn.testin.business.interfaces.user.IUserActivityService`
> 业务：用户活跃度统计查询、活跃记录新增、活跃记录列表查询。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|---|---|---|---|
| 1 | GET | `/v3/user/user_activity/get_user_activity` | getUserActivityInfo | 获取用户活跃量统计（分页） |
| 2 | POST | `/v3/user/user_activity/add_user_activity` | addUserActivity | 新增用户活跃记录 |
| 3 | GET | `/v3/user/user_activity/get_user_activity_list` | getUserActivityList | 查询用户活跃记录列表（分页） |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`。
分页默认值：`page = Constants.PAGE_DEFAULT`，`pageSize = Constants.PAGE_SIZE_DEFAULT`。
GET 接口带 `@UnderlineToCamel`：下划线 query 参数自动转驼峰。

涉及表：`user_activity`（`UserActivity`），底层通过 `IUserActivityService` 操作。

---

## 1. GET /v3/user/user_activity/get_user_activity — 用户活跃量统计

### 入口

`UserActivityController.getUserActivityInfo(@UnderlineToCamel UserActivityRequestDTO request)`

### 请求参数（Query，UserActivityRequestDTO，下划线转驼峰）

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| page | Integer | 否 | `PAGE_DEFAULT` | 页码（<=0 自动修正） |
| page_size | Integer | 否 | `PAGE_SIZE_DEFAULT` | 每页条数（<=0 自动修正） |
| start_date | String | 否 | — | 开始时间 |
| end_date | String | 否 | — | 结束时间 |
| activity_type | Integer | 否 | — | 活跃类型：1=日活跃，2=月活跃 |

### 响应结构

`ResponseResult<BasePageListResponseDTO<UserActivityResponseDTO>>`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 活跃统计列表（UserActivityResponseDTO） |
| data.list[].activityDate | String | 用户活跃日期 |
| data.list[].activityNumber | Integer | 用户活跃数量 |
| data.page / pageSize / totalPage / totalRow | Integer | 分页信息 |

### 实现意图

统计各用户的活跃量数据，支持多维筛选和分页。

### 调用链

```
UserActivityController.getUserActivityInfo
└─ IUserActivityService.getUserActivityInfo
```

### 关联横切

- `@UnderlineToCamel`：下划线 query 参数自动转驼峰。

---

## 2. POST /v3/user/user_activity/add_user_activity — 新增活跃记录

### 入口

`UserActivityController.addUserActivity(@RequestBody UserActivity userActivity)`

### 请求参数（JSON Body，UserActivity）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Integer | 否 | 主键 ID（自增，无需传） |
| userId | Integer | 否 | 用户 ID |
| activeDate | String | 否 | 活跃日期（`yyyy-MM-dd` 格式，LocalDate） |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 新增记录的主键 ID。

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Long | 新增记录的主键 ID |

### 实现意图

记录一条用户活跃事件，返回新增记录 ID。

### 调用链

```
UserActivityController.addUserActivity
└─ IUserActivityService.addUserActivity
```

---

## 3. GET /v3/user/user_activity/get_user_activity_list — 活跃记录列表

### 入口

`UserActivityController.getUserActivityList(requestDate, activityType, page, pageSize)`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| request_date | String | 是 | — | 查询日期（格式如 `yyyy-MM-dd`） |
| activity_type | Integer | 是 | — | 活动类型 |
| page | Integer | 否 | `PAGE_DEFAULT` | 页码（<=0 自动修正） |
| page_size | Integer | 否 | `PAGE_SIZE_DEFAULT` | 每页条数（<=0 自动修正） |

### 响应结构

`ResponseResult<BasePageListResponseDTO<UserActivityInfoDTO>>`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | 活跃记录列表（UserActivityInfoDTO） |
| data.list[].userId | Integer | 用户 ID |
| data.list[].account | String | 账号 |
| data.list[].userName | String | 姓名 |
| data.list[].mobile | String | 手机号码 |
| data.list[].position | String | 职位 |
| data.list[].role | String | 角色 |
| data.page / pageSize / totalPage / totalRow | Integer | 分页信息 |

### 实现意图

按日期和活动类型查询用户活跃记录明细列表，分页返回。

### 调用链

```
UserActivityController.getUserActivityList
└─ IUserActivityService.getUserActivityList
```
