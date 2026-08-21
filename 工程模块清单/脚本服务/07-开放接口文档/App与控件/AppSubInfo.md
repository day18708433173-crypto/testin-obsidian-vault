# AppSubInfo (ApiServlet)
> 包路径：cn.testin.service.app.AppSubInfo
> 调用方式：action=app op=AppSubInfo.{method}

## 职责
App 子信息管理服务，提供 App 子信息的增删改查功能，用于管理 App 的附加属性和配置信息。

## 方法列表

### list
查询 App 子信息列表（分页）。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业ID |
| appName | String | 否 | App 名称 |
| packageName | String | 否 | 包名 |
| page | Integer | 否 | 页码，默认1 |
| pageSize | Integer | 否 | 每页大小，默认20 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | App 子信息列表，元素为 AppSubInfo |
| data.page | Integer | 当前页码 |
| data.pageSize | Integer | 每页大小 |
| data.totalRow | Integer | 总行数 |
| data.totalPage | Integer | 总页数 |

### add
添加 App 子信息。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业ID |
| appName | String | 是 | App 名称 |
| appSubName | String | 是 | 子信息名称 |
| descr | String | 否 | 描述 |
| thirdUserId | String | 否 | 第三方用户ID |
| appId | String | 否 | App ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 新增结果 |

### getSubInfo
获取单条 App 子信息。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| appId | String | 是 | App ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.object | Object | App 子信息（AppSubInfo） |

### modifySubInfo
修改 App 子信息。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业ID |
| appName | String | 是 | App 名称 |
| appSubName | String | 是 | 子信息名称 |
| descr | String | 否 | 描述 |
| thirdUserId | String | 否 | 第三方用户ID |
| appId | String | 否 | App ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 更新结果 |

### delSubInfoById
根据 ID 删除 App 子信息。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业ID |
| appId | String | 否 | App ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 删除结果 |
