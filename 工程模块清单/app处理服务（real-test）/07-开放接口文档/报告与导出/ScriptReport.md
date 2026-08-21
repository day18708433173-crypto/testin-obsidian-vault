---
branch: syy.release.z7.8.1.0
module: real-test
type: 接口文档
entry: ApiServlet
---

# ScriptReport (service/report)

脚本报告检查信息的 ApiServlet。

类路径：`real-test/src/main/java/cn/testin/service/report/ScriptReport.java`，继承 `GenericBaseService`。

## 本类接口一览

| 接口 | op | 功能 |
| --- | --- | --- |
| checkInfos | ScriptReport.checkInfos | 获取脚本报告的检查信息列表 |

## checkInfos (`ScriptReport.checkInfos`)

- **实现意图**：根据 taskId 和 subtaskId 查询脚本报告的检查信息（如脚本检测结果、版本兼容性检查等）。

- **请求参数**：
| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| taskid | String | 否 | 任务 ID（与 skey 至少一个，均 blank 返回 10005） |
| skey | String | 否 | 分享链接 key |
| userid | Integer | 否 | 用户 ID |
| eid | Integer | 否 | 企业 ID |
| userprojectids | JSONArray | 否 | 用户项目组 ID 数组（元素 Integer） |
| subtaskid | String | 否 | 子任务 ID |
| subsubtaskid | String | 否 | 子子任务 ID |
| page | Integer | 否 | 页码，默认 1 |
| pageSize | Integer | 否 | 每页条数，默认 300 |

- **返回参数**：
| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 返回码，0 成功，非 0 失败 |
| msg | String | 返回信息 |
| data | JSONObject | 业务数据 |
| data.list | JSONArray | 检查信息列表，元素为 `PmrealCheckInfoSummary` |
| data.list[].id | String | MongoDB 产生的 ID |
| data.list[].taskid | String | 任务 ID |
| data.list[].subtaskid | String | 子任务 ID |
| data.list[].subsubtaskid | String | 子子任务 ID |
| data.list[].scriptid | Integer | 脚本 ID |
| data.list[].scriptNo | Integer | 脚本编号 |
| data.list[].scriptDescr | String | 脚本描述 |
| data.list[].orderNum | Integer | 脚本序号 |
| data.list[].stepid | Integer | 步骤 ID |
| data.list[].stepDescr | String | 步骤描述 |
| data.list[].name | String | 检查点名称 |
| data.list[].resultCategory | Integer | 检查点执行状态 |
| data.list[].errorMsg | String | 检查点错误信息 |
| data.list[].rule | String | 终止策略 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页条数 |
| data.totalRow | Integer | 总记录数 |
| data.totalPage | Integer | 总页数 |

- **调用链**：`ScriptReport` -> `IScriptReportService` -> `IPmrealCheckInfoDAO`。

- **涉及表与 SQL**：`pmreal_check_info`（MongoDB）。
