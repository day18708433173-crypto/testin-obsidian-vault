---
branch: syy.release.z7.8.1.0
module: real-test
type: 接口文档
entry: ApiServlet
---

# Stat (service/report)

统计分析异常查询的 ApiServlet。

类路径：`real-test/src/main/java/cn/testin/service/report/Stat.java`，继承 `GenericBaseService`。

## 本类接口一览

| 接口 | op | 功能 |
| --- | --- | --- |
| exceptions | Stat.exceptions | 查询任务异常统计信息 |

## exceptions (`Stat.exceptions`)

- **实现意图**：根据 taskId 查询任务的异常统计信息，包含异常类型分布、异常设备列表、异常脚本列表等。

- **请求参数**：
| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| taskid | String | 是 | 任务 ID（blank 返回 10005） |
| type | Integer | 否 | 异常类型：1 crash / 2 anr / 3 exception |
| subtaskid | String | 否 | 子任务 ID |
| deviceid | String | 否 | 设备 ID |
| page | Integer | 否 | 页码，默认 1（<1 时置 1） |
| pageSize | Integer | 否 | 每页条数，默认 100（范围 1~100） |

- **返回参数**：
| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 返回码，0 成功，非 0 失败 |
| msg | String | 返回信息 |
| data | JSONObject | 业务数据 |
| data.list | JSONArray | 异常列表，元素为 `Exceptions` |
| data.list[].taskid | String | 任务 ID |
| data.list[].name | String | 异常标题 |
| data.list[].content | String | 异常内容 |
| data.list[].num | Integer | 数量 |
| data.list[].startNum | Integer | 开始行数 |
| data.list[].endNum | Integer | 结束行数 |
| data.list[].deviceid | String | 设备 ID |
| data.list[].subtaskid | String | 子任务 ID |
| data.list[].subsubtaskid | String | 子子任务 ID |
| data.list[].logUrl | String | 日志地址 |
| data.list[].type | Integer | 日志类型：1 crash / 2 anr / 3 exception |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页条数 |
| data.totalRow | Integer | 总记录数 |
| data.totalPage | Integer | 总页数 |

- **调用链**：`Stat` -> `IPmrealStatExceptionDAO` -> MongoDB `pmreal_stat_exception`。

- **涉及表与 SQL**：`pmreal_stat_exception`（MongoDB）。
