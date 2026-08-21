---
branch: syy.release.z7.8.1.0
module: real-test
type: 接口文档
entry: WebMvc
---

# TemplateController

任务模板管理 MVC 控制器，提供模板的条件查询、批量删除、复制、定时任务转模板、更新模板内容、获取项目模板 ID 列表等功能。

类路径：`real-test/src/main/java/cn/testin/mvc/controller/TemplateController.java`，基础路径 `/v3/realtest/template`。

## 本类接口一览

| 接口 | 方法 | 路径 | 功能 |
| --- | --- | --- | --- |
| listByCondition | POST | /v3/realtest/template/templates | 条件分页查询模板列表 |
| batchRemove | DELETE | /v3/realtest/template/batch_delete | 批量删除模板 |
| copy | GET | /v3/realtest/template/copy | 复制定时任务为模板 |
| transTemplate | GET | /v3/realtest/template/trans_template | 定时任务转模板 |
| updateTemplateContent | POST | /v3/realtest/template/update_template_content | 更新模板内容 |
| listTemplateId | GET | /v3/realtest/template/list_template_id | 获取项目下模板 ID 列表 |

## listByCondition (`POST /v3/realtest/template/templates`)

- **实现意图**：根据 `TemplateRequestDTO` 条件分页查询模板列表。

- **请求参数**：body `TemplateRequestDTO`（继承 `PageRequestDTO`）。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| suiteId | Integer | 否 | 应用 ID |
| taskName | String | 否 | 任务名 |
| userIds | JSONArray | 否 | 用户 ID 列表（元素 Integer） |
| createStartTime | Long | 否 | 创建开始时间 |
| createEndTime | Long | 否 | 创建结束时间 |
| filterIds | JSONArray | 否 | 需要过滤的 ID（元素 Integer） |
| ids | JSONArray | 否 | 需要查询的 ID（元素 Integer） |
| plan | Boolean | 否 | 是否测试计划调用 |
| needContent | Boolean | 否 | 是否需要 content |
| needScriptDetail | Boolean | 否 | 查询是否需要脚本详细信息 |
| needBaseInfo | Boolean | 否 | 是否需要返回基础信息 |
| needScriptAndDeviceBashInfo | Boolean | 否 | 是否需要脚本和设备基础信息 |
| needDataSourceDetail | Boolean | 否 | 是否需要数据源相关信息 |
| checkStatus | Boolean | 否 | 是否检查执行状态 |
| page | Integer | 否 | 当前页 |
| pageSize | Integer | 否 | 页大小 |
| projectId | Integer | 否 | 项目 ID |

- **返回参数**：`ResponseResult<PageResponseDTO<TemplateResponseDTO>>`。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页条数 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总记录数 |
| data.list | JSONArray | 模板列表，元素为 `TemplateResponseDTO` |
| data.list[].taskId | Long | 任务 ID |
| data.list[].taskName | String | 任务名称 |
| data.list[].userName | String | 创建人 |
| data.list[].userId | Integer | 用户 ID |
| data.list[].createTime | Long | 创建时间 |
| data.list[].suiteId | Integer | 应用 ID |
| data.list[].isRelation | Integer | 是否关联 |
| data.list[].scriptTotal | Integer | 脚本数量（去重后） |
| data.list[].deviceTotal | Integer | 设备数量（去重后） |
| data.list[].content | String | 模板信息（用于提测） |
| data.list[].packageName | String | 包名 |
| data.list[].scriptNos | JSONArray | 脚本编号列表（元素 Integer，含脚本组展开） |
| data.list[].scriptRowInfo | JSONArray | 脚本行信息（元素 `ScriptResponse`，代码未确认字段） |
| data.list[].deviceIds | JSONArray | 设备 ID 列表（元素 String） |
| data.list[].jobRule | String | 定时策略 |
| data.list[].deviceResponse | JSONArray | 设备信息（元素 `DeviceResponse`，代码未确认字段） |
| data.list[].dataSourceResponse | JSONObject | 数据源信息（`DataSourceResponse`，代码未确认字段） |
| data.list[].sysPfId | Integer | 平台类型 |
| data.list[].taskType | Integer | 任务类型 |
| data.list[].envId | Integer | 环境 ID |
| data.list[].osType | Integer | 类型 |
| data.list[].appPackageId | Integer | app 包 ID |

- **调用链**：[TemplateService](../../../外部服务/TemplateService.md).listByCondition -> `IQuartzJobInfoDAO`（模板以定时任务形式存储）。
- **涉及表与 SQL**：`quartz_job_info`、`quartz_job_statement`。

## batchRemove (`DELETE /v3/realtest/template/batch_delete`)

- **实现意图**：批量删除指定模板（可能连带删除关联的定时任务记录）。

- **请求参数**：body `TemplateRemoveRequestDTO`。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| templateIds | JSONArray | 否 | 模板 ID 列表（元素 Long） |

- **返回参数**：`ResponseResult<Integer>`（删除影响行数）。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Integer | 删除影响行数 |

- **调用链**：[TemplateService](../../../外部服务/TemplateService.md).batchRemove -> `IQuartzJobInfoDAO` / `IQuartzJobStatementDAO`。

## copy (`GET /v3/realtest/template/copy`)

- **实现意图**：复制已有定时任务为新模板，供用户快速创建相似任务。
- **请求参数**：

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| job_id | Integer | 是 | 源定时任务 ID |
| user_id | Integer | 是 | 用户 ID |
| user_name | String | 是 | 用户名 |
| dir_id | Integer | 是 | 目标目录 ID |

- **返回参数**：`ResponseResult<BaseResponseDTO>`。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 操作影响行数 |

- **调用链**：[TemplateService](../../../外部服务/TemplateService.md).copyTemplateTask -> 读取源任务配置 -> 新建记录。

## transTemplate (`GET /v3/realtest/template/trans_template`)

- **实现意图**：将定时任务转为模板（标记任务为模板类型）。

- **请求参数**：

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| job_id | Integer | 是 | 定时任务 ID |

- **返回参数**：`ResponseResult<BaseResponseDTO>`。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 操作影响行数 |

- **调用链**：[TemplateService](../../../外部服务/TemplateService.md).jobTransferToTemplate -> 更新 `quartz_job_info` 类型标记。

## updateTemplateContent (`POST /v3/realtest/template/update_template_content`)

- **实现意图**：更新已有模板的内容配置（脚本、设备、参数等）。

- **请求参数**：body `TemplateContentRequestDTO`。

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| tagList | JSONArray | 否 | 标签 ID 列表（元素 Integer） |
| jobIdList | JSONArray | 否 | 模板 ID 列表（元素 Integer） |
| editType | Integer | 否 | 编辑类型（TagEditEnum） |

- **返回参数**：`ResponseResult<BaseResponseDTO>`。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data.result | Integer | 操作影响行数 |

- **调用链**：[TemplateService](../../../外部服务/TemplateService.md).updateTemplateContent -> 更新模板关联表。

## listTemplateId (`GET /v3/realtest/template/list_template_id`)

- **实现意图**：获取指定项目和业务编码下的可用模板 ID 列表。
- **请求参数**：

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| project_id | Integer | 是 | 项目 ID |
| biz_code | Integer | 是 | 业务编码 |

- **返回参数**：`ResponseResult<TemplateIdDTO>`。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data.totalCount | Integer | 模板总数 |
| data.templateIds | JSONArray | 模板 ID 列表（元素 Integer） |

- **调用链**：[TemplateService](../../../外部服务/TemplateService.md).listTemplateId。
