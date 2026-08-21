---
branch: syy.release.z7.8.1.0
module: real-test
type: 接口文档
entry: ApiServlet
---

# SpotReference (service/report)

抽查参考信息维护的 ApiServlet。

类路径：`real-test/src/main/java/cn/testin/service/report/SpotReference.java`，继承 `GenericBaseService`。

## 本类接口一览

| 接口 | op | 功能 |
| --- | --- | --- |
| maintain | SpotReference.maintain | 新增或更新抽查参考信息 |
| list | SpotReference.list | 分页查询抽查参考信息列表 |

## maintain (`SpotReference.maintain`)

- **实现意图**：新增或更新抽查参考信息（如参考截图、参考步骤、参考设备等），供质检抽查时作为对比基准。

- **请求参数**：
| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| projectid | Integer | 是 | 项目组 ID（null 或 <=0 返回 10005） |
| appid | Integer | 是 | 应用 ID（null 或 <=0 返回 10005） |
| name | String | 是 | 统计项名称（blank 返回 10005） |
| id | Integer | 否 | 自增主键，更新时传入 |
| standardtime | Double | 否 | 耗时达标值（<0 时置空） |
| challengetime | Double | 否 | 耗时挑战值 |
| standardUpNetwork | Double | 否 | 上行流量达标值 |
| challengeUpNetwork | Double | 否 | 上行流量挑战值 |
| standardDownNetwork | Double | 否 | 下行流量达标值 |
| challengeDownNetwork | Double | 否 | 下行流量挑战值 |

- **返回参数**：
| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 返回码，0 成功，非 0 失败 |
| msg | String | 返回信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 操作结果：1 成功，0 失败 |

- **调用链**：`SpotReference` -> `IRealSpotReferenceDAO`（MySQL）。

- **涉及表与 SQL**：`real_spot_reference`（INSERT/UPDATE）。

## list (`SpotReference.list`)

- **实现意图**：分页查询抽查参考信息列表，支持按 taskId、name 过滤。

- **请求参数**：
| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| taskid | String | 是 | 任务 ID（blank 抛 GeneralException） |
| name | String | 否 | 统计项名称，模糊过滤 |

- **返回参数**：
| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 返回码，0 成功，非 0 失败 |
| msg | String | 返回信息 |
| data | JSONObject | 业务数据 |
| data.list | JSONArray | 抽查参考信息列表，元素为 `RealSpotReference` |
| data.list[].id | Integer | 自增主键 |
| data.list[].projectid | Integer | 项目组 ID |
| data.list[].appid | Integer | 应用 ID |
| data.list[].name | String | 统计项名称 |
| data.list[].standardtime | Double | 耗时达标值 |
| data.list[].challengetime | Double | 耗时挑战值 |
| data.list[].standardUpNetwork | Double | 上行流量达标值 |
| data.list[].challengeUpNetwork | Double | 上行流量挑战值 |
| data.list[].standardDownNetwork | Double | 下行流量达标值 |
| data.list[].challengeDownNetwork | Double | 下行流量挑战值 |

- **调用链**：`IRealSpotReferenceDAO`（分页查询）。
