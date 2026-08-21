# Result (ApiServlet)
> 包路径：cn.testin.service.result.Result
> 调用方式：action=result op=Result.{method}

## 职责
测试执行结果管理服务，提供添加执行结果、解析结果数据、查询脚本结果统计、分页查询脚本测试结果等功能。

## 方法列表

### addResult
添加测试执行结果。

**请求参数（data字段）：** ScriptResult 实体字段
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| parentfileUrl | String | 是 | 父文件URL（为空则参数无效） |
| ... | ... | 否 | 其余 ScriptResult 实体字段 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Long | 新增结果（结果ID） |

### parse
解析结果数据。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| resultUrl | String | 是 | 结果文件URL |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码（结果文件中的状态码） |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.object | Object | 解析出的结果数据（原结果文件 data 字段被包装为 data.object） |

### getScriptResultCount
获取脚本结果统计数量。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| days | Integer | 是 | 统计天数 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 结果数量 |

### getScriptTestResultByPage
分页查询脚本测试结果。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| days | Integer | 是 | 查询天数 |
| startNo | Integer | 是 | 起始行号 |
| pageSize | Integer | 是 | 每页大小 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | 测试结果列表，元素为 ScriptResult |
