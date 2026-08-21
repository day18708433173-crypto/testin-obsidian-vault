# TabConfig (ApiServlet)
> 包路径：cn.testin.service.tab.TabConfig
> 调用方式：action=tab op=TabConfig.{method}

## 职责
Tab 配置管理服务，提供 Tab 标签页配置的增删改查功能，用于管理前端界面 Tab 页签的显示配置。

## 方法列表

### add
添加 Tab 配置。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业ID |
| projectid | Integer | 是 | 项目组ID |
| tabName | String | 是 | 页签名称 |
| urlUnique | String | 是 | URL 唯一标识 |
| creator | String | 否 | 创建人 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 新增结果 |

### delete
删除 Tab 配置。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| ids | JSONArray | 是 | 配置ID列表 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 删除结果 |

### list
查询 Tab 配置列表（分页）。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业ID |
| projectid | Integer | 是 | 项目组ID |
| keyWord | String | 否 | 关键字 |
| page | Integer | 否 | 页码，默认1 |
| pageSize | Integer | 否 | 每页大小，默认16 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | Tab 配置列表，元素为 TabConfigInfo |
| data.page | Integer | 当前页码 |
| data.pageSize | Integer | 每页大小 |
| data.totalRow | Integer | 总行数 |
| data.totalPage | Integer | 总页数 |

### update
更新 Tab 配置。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| id | Integer | 是 | 配置ID |
| urlUnique | String | 否 | URL 唯一标识 |
| tabName | String | 否 | 页签名称 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 更新结果 |

### get
获取单个 Tab 配置详情。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| id | Integer | 是 | 配置ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.object | Object | Tab 配置详情（TabConfigInfo） |
