# Suite (ApiServlet)
> 包路径：cn.testin.service.suite.Suite
> 调用方式：action=suite op=Suite.{method}

## 职责
测试套件管理服务，提供套件的增删改查、条件配置、App 绑定与解绑、图标管理、名称校验、关联脚本、按 ID 批量查询等功能。

## 方法列表

### add
新增测试套件。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业ID |
| projectid | Integer | 是 | 项目组ID |
| name | String | 是 | 套件名称 |
| userid | Integer | 是 | 用户ID |
| descr | String | 否 | 套件描述 |
| iconUrl | String | 否 | 套件图标URL |
| userName | String | 否 | 用户名 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 新增结果（成功时>0） |

### delete
删除测试套件。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| suiteId | Integer | 是 | 套件ID |
| userid | Integer | 否 | 用户ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 删除结果 |

### list
查询测试套件列表（分页）。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| suiteId | Integer | 否 | 套件ID |
| suiteIds | JSONArray | 否 | 套件ID列表 |
| name | String | 否 | 套件名称 |
| orderKey | String | 否 | 排序字段，默认 createtime |
| page | Integer | 否 | 页码，默认1 |
| pageSize | Integer | 否 | 每页大小，默认1000 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | 套件列表，元素为 SuiteInfo |
| data.page | Integer | 当前页码 |
| data.pageSize | Integer | 每页大小 |
| data.totalRow | Integer | 总行数 |
| data.totalPage | Integer | 总页数 |

### modify
修改测试套件信息。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| id | Integer | 是 | 套件ID |
| name | String | 否 | 套件名称 |
| descr | String | 否 | 套件描述（不传表示清空） |
| iconUrl | String | 否 | 套件图标URL（不传表示清空） |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 更新结果 |

### suiteCondition
查询套件条件列表（应用信息搜索条件）。

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
| data.list | JSONArray | 套件条件列表，元素为 SuiteInfo |

### get
获取单个套件详情。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| suiteId | Integer | 是 | 套件ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.object | Object | 套件详情（SuiteInfo） |

### deteteApp
套件解绑 App。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| suiteId | Integer | 是 | 套件ID |
| pkgid | Integer | 是 | 包ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 解绑结果 |

### apps
查询套件已绑定的 App 列表。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| suiteId | Integer | 是 | 套件ID |
| syspfId | Integer | 否 | 系统平台ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | App 列表，元素为 SuiteAppCondition |

### listApps
分页查询套件下可绑定的 App 列表。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| suiteId | Integer | 是 | 套件ID |
| appid | Integer | 否 | 应用ID |
| appVersion | String | 否 | 应用版本 |
| channelId | String | 否 | 渠道ID |
| osType | Integer | 否 | 系统类型 |
| packageName | String | 否 | 包名 |
| userIds | JSONArray | 否 | 用户ID列表 |
| page | Integer | 否 | 页码，默认1 |
| pageSize | Integer | 否 | 每页大小，默认15 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | App 列表，元素为 SuitePackageFile |
| data.page | Integer | 当前页码 |
| data.pageSize | Integer | 每页大小 |
| data.totalRow | Integer | 总行数 |
| data.totalPage | Integer | 总页数 |

### bind
套件绑定 App。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业ID |
| projectid | Integer | 是 | 项目组ID |
| appName | String | 是 | 应用名称 |
| packageName | String | 是 | 包名 |
| pkgid | Integer | 是 | 包ID |
| userid | Integer | 是 | 用户ID |
| userName | String | 否 | 用户名 |
| suiteId | Integer | 否 | 套件ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 绑定结果 |

### deleteIcon
删除套件图标。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| suiteId | Integer | 是 | 套件ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 删除结果 |

### checkSuiteName
校验套件名称是否可用。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| name | String | 是 | 套件名称 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.result | Integer | 校验结果（0=可用，>0=已存在） |

### associate
关联 App 到套件。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| appName | String | 是 | 应用名称 |
| packageName | String | 是 | 包名 |
| pkgid | Integer | 是 | 包ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.object | Object | 套件信息（SuiteInfo） |

### getSuiteApp
获取套件关联的 App 绑定关系。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 是 | 项目组ID |
| pkgid | Integer | 是 | 包ID |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.object | Object | App 绑定关系（SuiteApp） |

### getSuiteListByIds
根据 ID 列表批量查询套件。

**请求参数（data字段）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| suiteIds | JSONArray | 是 | 套件ID列表 |

**返回参数：**
| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0=成功 |
| message | String | 提示信息 |
| data | Object | 返回数据 |
| data.list | JSONArray | 套件列表，元素为 SuiteInfo |
