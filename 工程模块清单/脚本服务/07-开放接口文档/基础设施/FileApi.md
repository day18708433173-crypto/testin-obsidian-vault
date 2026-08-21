# FileApi (ApiServlet)
> 包路径：cn.testin.service.file.FileApi
> 调用方式：action=file op=FileApi.{method}

## 职责
文件通用接口服务，提供根据 ID 列表查询文件详情、清理文件缓存等功能。

## 方法列表

### listFileByIds
根据文件 ID 列表批量查询文件信息。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| fileIds | JSONArray | 是 | 文件ID列表 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | 文件信息列表，元素为 CommonFile |

### cleanCacheByFileId
根据文件 ID 清理文件缓存。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| fileId | Integer | 是 | 文件ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 被清理缓存的文件ID |

## 外部服务依赖
- **FileUpload**：文件存储与缓存管理
