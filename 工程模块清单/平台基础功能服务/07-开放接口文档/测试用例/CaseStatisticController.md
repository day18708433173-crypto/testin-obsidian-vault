# CaseStatisticController — 测试用例统计

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/testCase/CaseStatisticController.java`
> 类级路由：`/v3/test_case/case_statistic`
> Service 实现：`cn.testin.business.impl.testCase.CaseStatisticServiceServiceImpl`（约 264 行，委托给 `ICaseStatisticService`）
> 业务：用例统计信息的周维度查询、自动化/非自动化用例数量统计、新增/更新趋势统计、CSV 导出。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 | 操作日志 |
|---|---|---|---|---|---|
| 1 | GET | `/v3/test_case/case_statistic/list_case_statistic` | listCaseStatistic | 分页查询用例周统计列表 | 无 |
| 2 | GET | `/v3/test_case/case_statistic/case_statistic_info` | getCaseStatisticInfo | 用例总量与自动化用例数量统计 | 无 |
| 3 | GET | `/v3/test_case/case_statistic/case_statistic_view` | getCaseStatisticView | 新增/更新/执行统计视图 | 无 |
| 4 | GET | `/v3/test_case/case_statistic/export` | export | 导出用例统计 CSV | 无 |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`。导出接口为 `void`，直接写 `HttpServletResponse` 流。

---

## 1. GET /v3/test_case/case_statistic/list_case_statistic — 用例统计列表

### 入口

`CaseStatisticController.listCaseStatistic(CaseStaticRequestDTO requestDTO)`（`@UnderlineToCamel`）

### 请求参数（CaseStaticRequestDTO，Query）

| 字段 | 必填 | 说明 |
|---|---|---|
| project_id | 是 | 项目 ID |
| eid | 否 | 企业 ID |
| page | 否 | 页码，缺省 `PAGE_DEFAULT` |
| pageSize | 否 | 每页条数，缺省 `PAGE_SIZE_THIRTY`（30） |
| orderByType | 否 | 排序规则 |

### 响应结构

`ResponseResult<BasePageListResponseDTO<CaseStatisticResponse>>`。

### 返回参数（CaseStatisticResponse 元素结构）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 分页数据对象 |
| data.list | Array\<CaseStatisticResponse\> | 周统计列表 |
| data.list[].id | Integer | 主键 |
| data.list[].projectId | Integer | 项目 ID |
| data.list[].yearMonthWeek | String | 年月周标识（如 `2026-W32`） |
| data.list[].caseTotal | Integer | 用例总量 |
| data.list[].autoCaseTotal | Integer | 自动化用例数 |
| data.list[].noAutoCaseTotal | Integer | 非自动化用例数 |
| data.list[].startTime | LocalDate | 统计周期起始日期 |
| data.list[].endTime | LocalDate | 统计周期结束日期 |
| data.list[].statisticTime | Long | 统计生成时间 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页条数 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总记录数 |

### 实现意图

分页查询 `db_case.case_statistic` 表，按 `projectId` 过滤，支持排序规则。数据由定时任务 `caseStatistic()` 每周生成。

### 涉及表

- `db_case.case_statistic`

---

## 2. GET /v3/test_case/case_statistic/case_statistic_info — 自动化用例统计

### 入口

`CaseStatisticController.getCaseStatisticInfo(CaseStaticRequestDTO requestDTO)`（`@UnderlineToCamel`）

### 请求参数

| 字段 | 必填 | 说明 |
|---|---|---|
| project_id | 是 | 项目 ID |
| eid | 否 | 企业 ID |

### 响应结构

`ResponseResult<CaseStatisticInfoDTO>`。

### 返回参数（CaseStatisticInfoDTO）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.autoCaseTotal | Integer | 自动化用例数量 |
| data.caseTotal | Integer | 用例总量 |

### 实现意图

实时统计：查 `case_info` 总条数 + 按 eid/projectId 分组统计关联了有效脚本的用例数（即自动化用例数）。

### 调用链

```
CaseStatisticController.getCaseStatisticInfo
└─ CaseStatisticServiceServiceImpl.getCaseStaticInfo
   ├─ ICaseInfoService.selectCaseCountCondition (db_case.case_info, 总量)
   └─ ICaseInfoService.selectCaseCountAutoCaseGroupByEidProjectId (自动化用例数)
```

### 涉及表

- `db_case.case_info`

---

## 3. GET /v3/test_case/case_statistic/case_statistic_view — 统计视图

### 入口

`CaseStatisticController.getCaseStatisticView(CaseStaticRequestDTO requestDTO)`（`@UnderlineToCamel`）

### 请求参数

| 字段 | 必填 | 说明 |
|---|---|---|
| project_id | 是 | 项目 ID |
| eid | 否 | 企业 ID |
| startTime | 否 | 起始时间戳（ms） |
| endTime | 否 | 结束时间戳（ms） |

### 响应结构

`ResponseResult<CaseStatisticViewResponse>`。

### 返回参数（CaseStatisticViewResponse）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 数据对象 |
| data.createCaseTotal | Integer | 新增用例数 |
| data.updateCaseTotal | Integer | 更新用例数 |
| data.executeCaseTotal | Integer | 执行用例数（来自实时任务服务） |
| data.successCaseTotal | Integer | 成功用例数 |
| data.failCaseTotal | Integer | 失败用例数 |
| data.skipCaseTotal | Integer | 跳过用例数 |
| data.cancelCaseTotal | Integer | 取消用例数 |

### 实现意图

分两路：①调用 `RealTaskV3Api.getCaseStatisticView` 获取执行维度统计 ②查 `case_info` 按 `createTime/updateTime` 区间分别统计新增和更新数量。

### 调用链

```
CaseStatisticController.getCaseStatisticView
└─ CaseStatisticServiceServiceImpl.getCaseStaticView
   ├─ RealTaskV3Api.getCaseStatisticView → [实时任务服务](../../../任务管理服务/00-首页.md)
   └─ ICaseInfoService.selectCaseCountCondition (db_case.case_info, 按createFlag区分新增/更新)
```

### 涉及表

- `db_case.case_info`

---

## 4. GET /v3/test_case/case_statistic/export — 导出 CSV

### 入口

`CaseStatisticController.export(CaseStaticRequestDTO requestDTO, HttpServletResponse response)`（`@UnderlineToCamel`）

### 请求参数

| 字段 | 必填 | 说明 |
|---|---|---|
| project_id | 是 | 项目 ID |
| orderByType | 否 | 排序规则，缺省 DESC |

### 响应

直接写入 `HttpServletResponse`，Content-Type: `application/octet-stream`，文件名格式：`{项目名}-用例总量-{日期}.csv`。

CSV 表头：`年月周, 日期范围, 用例总量, 自动化用例数, 非自动化用例数`。含 BOM（`StatisticsConstant.CSV_BOM`），逐页（每页 100 条）流式写入。

### 调用链

```
CaseStatisticController.export
└─ CaseStatisticServiceServiceImpl.export
   ├─ ProjectService.get (查项目名称)
   ├─ ICaseStatisticDAO.listCaseStatisticCondition (db_case.case_statistic, 分页循环)
   └─ OutputStreamWriter → HttpServletResponse
```

### 涉及表

- `db_case.case_statistic`

