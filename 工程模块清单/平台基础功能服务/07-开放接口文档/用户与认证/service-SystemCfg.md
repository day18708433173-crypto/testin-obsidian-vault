# service-SystemCfg — 系统配置接口（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/user/SystemCfg.java`
> 基础类：`cn.testin.service.GenericBaseService`
> 机制：请求走 `ApiServlet /*` 入口，通过 `action=user` + `op=SystemCfg.方法名` 路由到此类的对应 public 方法；每个方法的参数为 `ApiRequest`，返回 JSON 字符串。
> - **action**: `user`（对应包 `cn.testin.service.user`）
> - **入口格式**：`{"op": "SystemCfg.方法名", "action": "user", "data": {...}}`
> 业务：系统配置的查询与维护、SSO 配置查询。

## op 列表总表

| # | op | 方法名 | 说明 |
|---|---|---|---|
| 1 | SystemCfg.getSystemConfig | getSystemConfig | 获取系统配置信息 |
| 2 | SystemCfg.maintain | maintain | 维护系统配置项（按 key 批量更新 system-config 组） |
| 3 | SystemCfg.sso | sso | 查询 SSO 配置（可按 appId 筛选） |

---

## 1. op=SystemCfg.getSystemConfig — 获取系统配置

### 请求格式
{"op": "SystemCfg.getSystemConfig", "action": "user", "data": {}}

### 请求参数

无。

### 核心逻辑

从 `ISystemService.getSystemConfig()` 读取系统级配置（封装为 `DbSystemConfig`），返回前端需要的一系列配置值。

### 响应

```json
{ error_code, msg, data: { object: {
  itestinProUrl, itestinProVersion, deletedFileExpireTime, validResultExpireTime,
  updateTime, indexUrl, createTaskUrl, forgotPwdUrl,
  uploadFileSize, uploadFileExt, uploadFileUrl,
  webSystemVersion, groupAi, groupCount
} } }
```

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.object | Object | 系统配置对象（无配置时为空对象） |
| data.object.itestinProUrl | String | iTestin 客户端下载地址 |
| data.object.itestinProVersion | String | iTestin 客户端版本 |
| data.object.deletedFileExpireTime | Long | 已删除文件物理保存时长（秒） |
| data.object.validResultExpireTime | Long | 有效结果文件过期时间（秒） |
| data.object.updateTime | Long | 最后更新时间 |
| data.object.indexUrl | String | 前台首页地址 |
| data.object.createTaskUrl | String | 创建任务页面地址 |
| data.object.forgotPwdUrl | String | 忘记密码地址 |
| data.object.uploadFileSize | Integer | 上传文件大小（M） |
| data.object.uploadFileExt | String | 上传文件扩展名 |
| data.object.uploadFileUrl | String | 上传文件地址 |
| data.object.webSystemVersion | String | Web 系统版本 |
| data.object.groupAi | String | 多机联动 AI 开关 |
| data.object.groupCount | String | 多机联动设备数量 |

### 涉及表

- `db_system_config`（`DbSystemConfig`）— 系统配置表

---

## 2. op=SystemCfg.maintain — 维护系统配置

### 请求格式
{"op": "SystemCfg.maintain", "action": "user", "data": {"id": ..., "itestinProUrl": "...", ...}}

### 请求参数

`DbSystemConfig` JSON 对象，`id` 必填（`systemConfig.getId() == null` 抛参数异常）。传入非空字段将逐条更新 `system-config` 组下的对应 `system_param`。

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Integer | 是 | 配置记录 ID |
| itestinProUrl | String | 否 | iTestin 客户端下载地址 |
| itestinProVersion | String | 否 | iTestin 客户端版本 |
| validResultExpireTime | Long | 否 | 有效结果文件过期时间（秒） |
| deletedFileExpireTime | Long | 否 | 已删除文件物理保存时长（秒） |
| indexUrl | String | 否 | 前台首页地址 |
| createTaskUrl | String | 否 | 创建任务页面地址 |
| forgotPwdUrl | String | 否 | 忘记密码地址 |
| uploadFileUrl | String | 否 | 上传文件地址 |
| uploadFileSize | Integer | 否 | 上传文件大小（M） |
| uploadFileExt | String | 否 | 上传文件扩展名 |
| webSystemVersion | String | 否 | Web 系统版本 |
| groupAi | String | 否 | 多机联动 AI 开关 |
| groupCount | String | 否 | 多机联动设备数量 |

支持更新的字段映射：

| 字段 | paramKey |
|---|---|
| itestinProUrl | `itestin_pro_url` |
| itestinProVersion | `itestin_pro_version` |
| validResultExpireTime | `valid_result_expire_time` |
| deletedFileExpireTime | `deleted_file_expire_time` |
| indexUrl | `index_url` |
| createTaskUrl | `create_task_url` |
| forgotPwdUrl | `forgot_pwd_url` |
| uploadFileUrl | `upload_file_url` |
| uploadFileSize | `upload_file_size` |
| uploadFileExt | `upload_file_ext` |
| webSystemVersion | `web_system_version` |
| groupAi | `group_ai` |
| groupCount | `group_count` |

### 核心逻辑

遍历传入的非空字段，调用 `isystemservice.updateSystemParam("system-config", key, value)` 逐项更新。

### 响应

`{ error_code, msg, data: { result: 1 } }` — 数据不存在时抛 `noneData`，更新失败抛 `sqlException`。

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.result | Integer | 固定为 1 |

### 涉及表

- `db_system_param`（`DbSystemParam`）— 系统参数表（`system-config` 组）

---

## 3. op=SystemCfg.sso — 查询 SSO 配置

### 请求格式
{"op": "SystemCfg.sso", "action": "user", "data": {"appId": "..."}}

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| appId | String | 否 | SSO 应用 ID；传入则筛选该 app 的配置，否则返回全部（`isNull` 判断，为空时不筛选） |

### 核心逻辑

查询 `db_sso_config` 表。传 appId 走 `getSSOConfigByPool`，否则走 `getSSOConfig`。

### 响应

`{ error_code, msg, data: { list: [DbSSOConfig, ...] } }`

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| error_code | Integer | 错误码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.list | Array | SSO 配置列表 |
| data.list[].id | Integer | 配置 ID |
| data.list[].appID | String | 应用 ID |
| data.list[].appKey | String | 应用密钥 |
| data.list[].loginUrl | String | 登录地址 |
| data.list[].userInfoUrl | String | 用户信息地址 |
| data.list[].logoutUrl | String | 登出地址 |
| data.list[].parseTool | String | 解析工具 |
| data.list[].callBackUrl | String | 回调地址 |
| data.list[].extraInfo | String | 扩展信息 |
| data.list[].active | Integer | 是否启用 |

### 涉及表

- `db_sso_config`（`DbSSOConfig`）— SSO 配置表
