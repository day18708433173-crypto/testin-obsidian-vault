# App (ApiServlet)
> 包路径：cn.testin.service.app.App
> 调用方式：action=app op=App.{method}

## 职责
App 管理服务，提供证书查询、根据项目查询 App 列表、查询所有包渠道、添加 App 等功能。与 UserManager（用户/项目查询）、SystemParamApi（系统参数）协作。

## 方法列表

### certificate
查询证书信息。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pkgid | Integer | 否 | 包ID（提供时优先使用） |
| appid | Integer | 否 | 应用ID |
| packageName | String | 否 | 包名（与 osType 配合） |
| osType | Integer | 否 | 系统类型（与 packageName 配合） |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.object | Object | 证书信息（signfileurl/keypass/storepass/alias/commonCertificate） |

### findAppsByProjectId
根据项目 ID 分页查询 App 列表。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| page | Integer | 是 | 页码 |
| pageSize | Integer | 是 | 每页大小 |
| syspfId | Integer | 否 | 系统平台ID |
| suiteId | Integer | 否 | 套件ID |
| isUniquePackageName | Boolean | 否 | 是否按包名去重，默认 false |
| osTypes | JSONArray | 否 | 系统类型列表 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | App 列表，元素为 AppInfo |
| data.page | Integer | 当前页码 |
| data.pageSize | Integer | 每页大小 |
| data.totalRow | Integer | 总行数 |
| data.totalPage | Integer | 总页数 |

### selectAllPackageChannels
查询所有包渠道列表。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| suiteId | Integer | 否 | 套件ID |
| appid | Integer | 否 | 应用ID |
| appVersion | String | 否 | 应用版本 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | 渠道列表，元素为 String |

### addApp
添加新的 App。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| fileUrl | String | 是 | 文件URL |
| packageName | String | 否 | 包名 |
| appName | String | 否 | App 名称 |
| appVerison | String | 否 | App 版本 |
| osType | String | 否 | 系统类型（android/ios/HarmonyOS） |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 固定为 1（成功） |

## 外部服务依赖
- **OnlineApi**：在线会话查询
- **SystemParamApi**：系统参数查询
