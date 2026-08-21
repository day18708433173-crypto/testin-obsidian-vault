---
branch: syy.release.z7.8.1.0
module: real-test
type: 接口文档
entry: ApiServlet
---

# Quartz (service/app)

定时任务辅助查询的 ApiServlet，提供应用名、渠道、Suite 名、版本等下拉列表数据。

类路径：`real-test/src/main/java/cn/testin/service/app/Quartz.java`，继承 `GenericBaseService`。

## 本类接口一览

| 接口 | op | 功能 |
| --- | --- | --- |
| appNameList | Quartz.appNameList | 获取项目下应用名列表 |
| appChannelList | Quartz.appChannelList | 获取应用渠道列表（@Deprecated） |
| listQuartzJobByNames | Quartz.listQuartzJobByNames | 根据名称列表查询定时任务 |
| suiteNameList | Quartz.suiteNameList | 获取项目下 Suite 名列表 |
| appVersionList | Quartz.appVersionList | 获取应用版本列表 |

## appNameList (`Quartz.appNameList`)

- **实现意图**：获取指定项目下所有已测试过的应用名列表（供搜索下拉框用）。

- **请求参数**：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| projectid | Integer | 是 | 项目组 ID（null 或 <=0 返回 paraInvalid） |
| bizCode | Integer | 否 | 业务编码（传值时须 >0） |
| suiteId | Integer | 否 | 应用 ID（跨平台） |
| syspfId | Integer | 否 | 系统平台 |
| jobIds | JSONArray | 否 | 定时任务 ID 列表（元素 Integer） |

- **返回参数**：`{code, msg, data}`，data 含 `list`。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 应用名列表，元素为 `DbQuartzJobInfo`（字段见下表） |

data.list 元素（`DbQuartzJobInfo`）关键字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| jobId | Long | 定时任务 ID |
| jobName | String | 定时任务名称 |
| jobRule | String | 定时任务规则 |
| jobStatus | String | 定时任务状态 |
| jobRemark | String | 备注 |
| userId | Integer | 用户 ID |
| userName | String | 用户名 |
| eid | Integer | 企业 ID |
| projectId | Integer | 项目组 ID |
| taskDesc | String | 任务描述 |
| appId | Integer | appId |
| appName | String | 应用名称 |
| appVersion | String | 应用版本 |
| pkgId | Integer | 包 ID |
| packageName | String | 包名称 |
| bizCode | Integer | 业务类型 |
| syspfId | Integer | 平台类型 |
| channelId | String | 应用渠道 ID |
| suiteId | Integer | 应用 ID（跨平台） |
| jobType | Integer | 任务类型（0 指定时间/1 定点/2 cron/3 高级） |
| dirId | Integer | 目录 ID |
| createTime | Long | 创建时间 |
| updateTime | Long | 修改时间 |

- **调用链**：`IQuartzJobInfoDAO`（去重查询应用名）。

- **涉及表与 SQL**：`quartz_job_info`（SELECT DISTINCT app_name）。

## appChannelList (`Quartz.appChannelList`)

- **实现意图**：获取指定项目的应用渠道列表（@Deprecated 已废弃）。

- **请求参数**：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| projectid | Integer | 是 | 项目组 ID（null 或 <=0 返回 paraInvalid） |
| bizCode | Integer | 是 | 业务编码（null 或 <=0 返回 paraInvalid） |
| suiteId | Integer | 否 | 应用 ID（跨平台） |
| appid | Integer | 否 | appId |
| appVersion | String | 否 | 应用版本 |

- **返回参数**：`{code, msg, data}`，data 含 `list`。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 渠道列表，元素 String（渠道名称） |

- **调用链**：`IQuartzJobInfoDAO`（`queryJobAppChannelList`）。

## listQuartzJobByNames (`Quartz.listQuartzJobByNames`)

- **实现意图**：根据定时任务名称列表批量查询任务详情。

- **请求参数**：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| jobNames | JSONArray | 是 | 定时任务名称列表（元素 String，null 或空返回 paraInvalid） |

- **返回参数**：`{code, msg, data}`，data 含 `list`。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 定时任务列表，元素为 `DbQuartzJobInfo`（字段同 appNameList） |

- **调用链**：`IQuartzJobInfoDAO`（`listQuartzJobByNames`）。

## suiteNameList (`Quartz.suiteNameList`)

- **实现意图**：获取项目下的 Suite（应用集）ID 列表。

- **请求参数**：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| projectid | Integer | 是 | 项目组 ID（null 或 <=0 返回 paraInvalid） |
| bizCode | Integer | 否 | 业务编码（传值时须 >0） |

- **返回参数**：`{code, msg, data}`，data 含 `list`。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | Suite ID 列表，元素 Integer |

- **调用链**：`IQuartzJobInfoDAO`（`querySuiteNames`）。

## appVersionList (`Quartz.appVersionList`)

- **实现意图**：获取指定应用的所有版本号列表。

- **请求参数**：

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| projectid | Integer | 是 | 项目组 ID（null 或 <=0 返回 paraInvalid） |
| bizCode | Integer | 是 | 业务编码（null 或 <=0 返回 paraInvalid） |
| suiteId | Integer | 否 | 应用 ID（跨平台） |
| syspfId | Integer | 否 | 系统平台 |
| appid | Integer | 否 | appId |

- **返回参数**：`{code, msg, data}`，data 含 `list`。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data.list | JSONArray | 版本号列表，元素 String |

- **调用链**：`IQuartzJobInfoDAO`（`queryJobAppVersionList`）。外部服务：[ScriptService](../../../脚本服务/00-首页.md)（应用版本查询）。
