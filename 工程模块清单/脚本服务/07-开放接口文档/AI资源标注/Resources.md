# Resources (ApiServlet)
> 包路径：cn.testin.service.ai.Resources
> 调用方式：action=ai op=Resources.{method}

## 职责
AI 数据标注资源管理服务，提供标注资源的增删改查、关联映射列表、导出和导入功能。用于管理 AI 训练所需的标注数据资源。

## 方法列表

### add
添加标注资源。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| userid | Integer | 是 | 用户ID |
| smallUrl | String | 是 | 小图URL（为空则参数无效） |
| suiteid | Integer | 否 | 套件ID |
| userprojectids | JSONArray | 否 | 用户项目ID列表（权限校验用） |
| ... | ... | 否 | 其余 ResourcesData 实体字段 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 新增结果 |
| apikey | String | 接口密钥（透传） |
| mkey | String | 模块key（透传） |

### update
更新标注资源。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | Integer | 是 | 资源ID |
| uid | Integer | 是 | 用户ID |
| markName | String | 是 | 标注名称 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 更新结果 |

### remove
删除标注资源。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| resourceIds | JSONArray | 是 | 资源ID列表 |
| suiteid | Integer | 否 | 套件ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 删除结果（-1 表示无数据） |

### list
查询标注资源列表（分页）。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| suiteid | Integer | 否 | 套件ID |
| sourceType | Integer | 否 | 来源类型，默认 APP |
| eid | Integer | 否 | 企业ID |
| projectid | Integer | 否 | 项目组ID |
| page | Integer | 否 | 页码，默认1 |
| pageSize | Integer | 否 | 每页大小，默认全部 |
| statuses | Integer | 否 | 状态，默认1 |
| startUpdatetime | Long | 否 | 起始更新时间 |
| fuzzyByMarkName | Integer | 否 | 是否模糊查询标注名（null=精准） |
| markName | String | 否 | 标注名称 |
| packageName | String | 否 | 包名（suiteid 有效时忽略） |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | 资源列表，元素为 ResourcesData |
| data.object | Object | 按标注名分组的资源 Map（markName → List\<ResourcesData\>） |
| data.totalRow | Integer | 总行数 |
| data.totalPage | Integer | 总页数 |
| data.page | Integer | 当前页码 |
| data.pageSize | Integer | 每页大小 |

### mapList
查询标注资源的关联映射列表（online 用）。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| suiteid | Integer | 否 | 套件ID |
| sourceType | Integer | 否 | 来源类型，默认 APP |
| eid | Integer | 否 | 企业ID |
| projectid | Integer | 否 | 项目组ID |
| page | Integer | 否 | 页码，默认1 |
| pageSize | Integer | 否 | 每页大小，默认全部 |
| statuses | Integer | 否 | 状态，默认1 |
| markName | String | 否 | 标注名称 |
| packageName | String | 否 | 包名（suiteid 有效时忽略） |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.object | Object | 关联映射 Map（markName → List\<ResourcesData\>） |
| data.totalRow | Integer | 总行数 |
| data.totalPage | Integer | 总页数 |
| data.page | Integer | 当前页码 |
| data.pageSize | Integer | 每页大小 |

### export
批量导出标注资源。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| ids | JSONArray | 是 | 资源ID列表 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.object | Object | 导出响应（ResponseData） |

### importResource
批量导入标注资源。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| url | String | 是 | 导入文件URL |
| uid | Integer | 否 | 用户ID |
| eid | Integer | 否 | 企业ID |
| projectid | Integer | 否 | 项目组ID |
| suiteid | Integer | 否 | 套件ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.object | Object | 导入响应（ImportResponse） |

## 外部服务依赖
- **FileUpload**：文件上传服务
- **ExportDispatchThread/ImportDispatchThread**：异步导出导入队列
