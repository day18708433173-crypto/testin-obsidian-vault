# CaseActionLogController — 用例操作日志

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/testCase/CaseActionLogController.java`
> 类级路由：`/v3/test_case/case_action_log`
> Service 实现：`cn.testin.business.impl.testCase.CaseActionLogServiceImpl`（约 297 行，委托给 `ICaseActionLogService`）
> 业务：查询用例的操作日志（新增、编辑、删除、步骤增删改移、online 保存等），并在日志详情中回填脚本名称。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 | 操作日志 |
|---|---|---|---|---|---|
| 1 | GET | `/v3/test_case/case_action_log` | getCaseActionLogList | 分页查询用例操作日志 | 无 |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`。

---

## 1. GET /v3/test_case/case_action_log — 获取操作日志

### 入口

`CaseActionLogController.getCaseActionLogList(@RequestParam Integer caseId, Integer page, Integer pageSize)`

### 请求参数（Query）

| 字段 | 必填 | 说明 |
|---|---|---|
| case_id | 是 | 用例 ID |
| page | 否 | 页码，缺省 `PAGE_DEFAULT` |
| page_size | 否 | 每页条数，缺省 `PAGE_SIZE_DEFAULT` |

### 响应结构

`ResponseResult<BasePageListResponseDTO<CaseActionLog>>`。

### 返回参数（CaseActionLog 元素结构）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 分页数据对象 |
| data.list | Array\<CaseActionLog\> | 操作日志列表 |
| data.list[].id | Integer | 日志主键 |
| data.list[].caseId | Integer | 用例 ID |
| data.list[].type | Integer | 操作类型（见 `CaseActionLogTypeEnum`：ADD/DELETE/ADD_STEP/REMOVE_STEP/MODIFY_STEP_INFO/ADJUST_STEP_ORDER/ONLINE_SAVE_STEPS 等） |
| data.list[].changeDetailsJson | String | 变更详情 JSON（before/after 步骤列表，查询时会回填脚本名称） |
| data.list[].createUserId | Integer | 操作人 ID |
| data.list[].createTime | Date | 创建时间 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页条数 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总记录数 |

### 实现意图

按 `caseId` 分页查询 `db_case.case_action_log`（按 `create_time` 倒序）。查询后根据操作类型（`CaseActionLogTypeEnum`）对日志内容做增强处理：

- **ADD / DELETE**（用例新增/删除）：解析 `changeDetailsJson` 中关联的步骤列表，调用 `ScriptV3Api.getCaseStepScriptList` 批量回填脚本名称。
- **ADD_STEP / REMOVE_STEP / MODIFY_STEP_INFO / ADJUST_STEP_ORDER / ONLINE_SAVE_STEPS**（步骤相关操作）：解析 before/after 中的 `CaseStepDTO` 列表，收集所有 `scriptNo` 并查脚本服务回填 `scriptName`。

### 调用链

```
CaseActionLogController.getCaseActionLogList
└─ CaseActionLogServiceImpl.getCaseActionLogList
   ├─ ICaseActionLogDAO.selectCount / selectList (db_case.case_action_log, limit分页)
   └─ case ActionLogTypeEnum switch → updateScriptNamesFor*()
      └─ ScriptV3Api.getCaseStepScriptList → [脚本服务](../../../脚本服务/00-首页.md) (回填脚本名称)
```

### 涉及表

- `db_case.case_action_log`

### 异常处理

- `caseId` 为空 → 返回空分页结果（`totalPage=0`）
- 查脚本服务失败时仅记录 `errorLog`，不阻断日志返回

