# AppPackage (ApiServlet)
> 包路径：cn.testin.service.app.AppPackage
> 调用方式：action=app op=AppPackage.{method}

## 职责
App 包文件管理服务，提供包文件列表查询、版本列表、包文件获取（按 ID 或 MD5）、签名信息补全、项目下 App 列表、获取单个及删除等功能。

## 方法列表

### listPackageFile
查询包文件列表（分页）。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| startPageNo | Integer | 否 | 起始页码，默认1 |
| pageSize | Integer | 否 | 每页大小，默认15 |
| appId | Integer | 否 | 应用ID |
| appVersion | String | 否 | 应用版本 |
| channel | String | 否 | 渠道 |
| osType | Integer | 否 | 系统类型 |
| projectId | Integer | 否 | 项目组ID |
| packageName | String | 否 | 包名 |
| appName | String | 否 | 应用名称 |
| userIds | JSONArray | 否 | 用户ID列表 |
| suiteId | Integer | 否 | 套件ID |
| packageId | Integer | 否 | 包ID |
| appPackageIds | JSONArray | 否 | 包ID列表 |
| osTypes | JSONArray | 否 | 系统类型列表 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | 包文件列表，元素为 SuitePackageFile |
| data.page | Integer | 当前页码 |
| data.pageSize | Integer | 每页大小 |
| data.totalRow | Integer | 总行数 |
| data.totalPage | Integer | 总页数 |

### listAppVersion
查询 App 版本列表。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| appId | Integer | 是 | 应用ID |
| projectId | Integer | 是 | 项目组ID |
| appVersion | String | 否 | 应用版本 |
| suiteId | Integer | 否 | 套件ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | 版本列表，元素为 PackageFile |

### getPackageFile
根据包 ID 或 MD5 获取包文件详情。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| packageId | Integer | 否 | 包ID（与 appMd5 至少一个有效） |
| appMd5 | String | 否 | 文件MD5（与 packageId 至少一个有效） |
| packageName | String | 否 | 包名 |
| projectid | Integer | 否 | 项目组ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.object | Object | 包文件详情（AppDetailDTO） |

### getPackageFileByAppMd5
根据 App 的 MD5 值获取包文件下载链接。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| appMd5 | String | 是 | 文件MD5 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.object | Object | 下载信息 |
| data.object.url | String | App 下载链接 |

### completeSingInfo
补全签名信息。

**请求参数（data字段）：** AppSignInfoDTO 实体字段
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| ... | ... | 否 | AppSignInfoDTO 实体字段 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.object | Object | 返回对象 |
| data.object.result | Integer | 1=成功，-1=失败 |

### listAppByProjectid
根据项目 ID 查询 App 列表。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | App 列表，元素为 AppInfo |

### get
获取单个 App 包信息。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pkgid | Integer | 是 | 包ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.object | Object | 包信息（PackageExt） |

### remove
删除 App 包。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pkgid | Integer | 是 | 包ID |
| suiteId | Integer | 是 | 套件ID |
| type | Integer | 否 | 删除类型 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 固定为 1（成功） |

## 外部服务依赖
- **FileUpload**：文件上传与解析
