# FormResource (ApiServlet)
> 包路径：cn.testin.service.form.FormResource
> 调用方式：action=form op=FormResource.{method}

## 职责
表单资源管理服务，提供表单资源的增删改查功能，用于管理脚本相关的表单模板和资源。

## 方法列表

### add
添加表单资源。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业ID |
| projectid | Integer | 是 | 项目组ID |
| name | String | 是 | 资源名称 |
| type | Integer | 是 | 资源类型（须为 FormResourceEnums 中定义值） |
| target | String | 是 | 目标值 |
| aiType | Integer | 否 | AI类型（区分 AI 模板与表单标注管理） |
| fieldName | String | 否 | 字段名 |
| formPlaceholder | String | 否 | 占位提示 |
| creator | String | 否 | 创建人 |
| formType | Integer | 否 | 表单类型 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 新增结果（-1 表示名称已存在） |

### delete
删除表单资源。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| eid | Integer | 是 | 企业ID |
| id | Integer | 是 | 资源ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 删除结果 |

### list
查询表单资源列表（分页）。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| eid | Integer | 是 | 企业ID |
| name | String | 否 | 资源名称 |
| type | Integer | 否 | 资源类型 |
| orderKey | String | 否 | 排序字段，默认 create_time |
| aiType | Integer | 否 | AI类型 |
| page | Integer | 否 | 页码，默认1 |
| pageSize | Integer | 否 | 每页大小，默认9999 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | 表单资源列表，元素为 FormResourceInfo |
| data.page | Integer | 当前页码 |
| data.pageSize | Integer | 每页大小 |
| data.totalRow | Integer | 总行数 |
| data.totalPage | Integer | 总页数 |

### update
更新表单资源。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | Integer | 是 | 资源ID |
| projectid | Integer | 是 | 项目组ID |
| eid | Integer | 是 | 企业ID |
| name | String | 否 | 资源名称 |
| type | Integer | 否 | 资源类型 |
| target | String | 否 | 目标值 |
| fieldName | String | 否 | 字段名 |
| formPlaceholder | String | 否 | 占位提示 |
| formType | Integer | 否 | 表单类型 |
| aiType | Integer | 否 | AI类型 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 更新结果（-1 表示名称已存在） |
