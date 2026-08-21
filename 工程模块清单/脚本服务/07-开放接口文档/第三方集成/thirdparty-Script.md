# Script（thirdparty） (ApiServlet)
> 包路径：cn.testin.service.thirdparty.Script
> 调用方式：action=thirdparty op=Script.{method}

## 职责
第三方集成脚本服务，提供第三方功能绑定/解绑、参数列表查询、按脚本编号查询、关联关系查询和复制等功能。用于与第三方系统（如接口用例平台）进行脚本和参数的联动。

## 方法列表

### bindThirdFunc
绑定第三方功能。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| scriptNo | Integer | 是 | 脚本编号 |
| thirdPartyScriptNo | String | 是 | 第三方脚本编号 |
| thirdPartyProjectid | String | 是 | 第三方项目ID |
| syncMessage | Integer | 否 | 是否同步消息 |
| thirdUserId | String | 否 | 第三方用户ID |
| thirdUserName | String | 否 | 第三方用户名 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 绑定结果 |

### unbind
解绑第三方功能。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| thirdPartyScriptNo | String | 是 | 第三方脚本编号 |
| thirdPartyProjectid | String | 是 | 第三方项目ID（或通过 onlineUserInfo 传入） |
| syncMessage | Integer | 否 | 是否同步消息 |
| thirdUserId | String | 否 | 第三方用户ID |
| thirdUserName | String | 否 | 第三方用户名 |
| onlineUserInfo | JSONObject | 否 | 在线用户信息（含 thirdPartyProjectid） |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 解绑结果 |

### paramList
查询参数列表。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| thirdPartyScriptNo | String | 是 | 第三方脚本编号 |
| projectid | Integer | 是 | 项目组ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | 参数列表，元素为 ScriptRelationParam |

### paramListByScriptNo
根据脚本编号查询参数列表。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| scriptNo | String | 是 | 脚本编号 |
| projectid | Integer | 是 | 项目组ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | 参数列表，元素为 ScriptRelationParam |

### getScriptByThirdPartyScriptNo
根据第三方脚本编号查询脚本。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| thirdPartyScriptNo | String | 是 | 第三方脚本编号 |
| projectid | Integer | 是 | 项目组ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.object | Object | 脚本信息（ScriptFile，未绑定时为空对象） |
| data.list | JSONArray | 脚本子脚本关联关系列表（ScriptRelation） |

### relationList
查询第三方关联关系列表。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| thirdPartyProjectid | String | 是 | 第三方项目ID |
| scriptNos | JSONArray | 否 | 脚本编号列表 |
| thirdPartyScriptNo | String | 否 | 第三方脚本编号 |
| thirdPartyUserName | String | 否 | 第三方用户名 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | 关联关系列表，元素为 ScriptThirdRelation |

### copyRelation
复制第三方关联关系。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| thirdPartyScriptNo | String | 是 | 目标第三方脚本编号 |
| originalThirdPartyScriptNo | String | 是 | 源第三方脚本编号 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 复制结果 |

## 外部服务依赖
- **InterfaceCase**：接口用例服务（平台基础功能服务 提供）
