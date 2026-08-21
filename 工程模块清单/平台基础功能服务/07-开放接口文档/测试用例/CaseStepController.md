# CaseStepController — 测试用例步骤

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/testCase/CaseStepController.java`
> 类级路由：`/v3/test_case/case_step`
> Service 实现：`cn.testin.business.impl.testCase.CaseStepServiceImpl`（约 338 行，委托给 `ICaseStepService`；同时依赖 `ICaseInfoService`）
> 业务：测试用例步骤的增删改查、排序移动、批量更新（online 模式）、关联脚本管理。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 | 操作日志 |
|---|---|---|---|---|---|
| 1 | GET | `/v3/test_case/case_step/case_steps` | getCaseStepList | 获取用例步骤列表（含脚本名称/状态） | 无 |
| 2 | POST | `/v3/test_case/case_step` | addCaseStep | 添加用例步骤 | `CASE_STEP_ADD` |
| 3 | PUT | `/v3/test_case/case_step` | editCaseStep | 更新用例步骤 | `CASE_STEP_UPDATE` |
| 4 | DELETE | `/v3/test_case/case_step/{step_id}` | deleteCaseStep | 删除用例步骤 | `CASE_STEP_DELETE` |
| 5 | POST | `/v3/test_case/case_step/move` | moveCaseStep | 用例步骤移动（调整排序） | `CASE_STEP_MOVE` |
| 6 | PUT | `/v3/test_case/case_step/case_steps` | updateCaseStep | 批量更新用例步骤（online 界面） | `CASE_STEP_UPDATE_ONLINE` |
| 7 | POST | `/v3/test_case/case_step/cancel_associated_script` | cancelScript | 取消步骤与脚本的关联 | `CASE_STEP_UPDATE` |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`；写操作返回 `BaseDataResultDTO { Long result }`。

---

## 1. GET /v3/test_case/case_step/case_steps — 获取步骤列表

### 入口

`CaseStepController.getCaseStepList(@RequestParam Integer caseId, Integer projectId)`

### 请求参数（Query）

| 字段 | 必填 | 说明 |
|---|---|---|
| case_id | 是 | 用例 ID |
| project_id | 否 | 项目 ID |

### 响应结构

`ResponseResult<List<CaseStepDTO>>`。

### 返回参数（CaseStepDTO 元素结构，继承 CaseStep）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Array\<CaseStepDTO\> | 步骤列表 |
| data[].id | Integer | 步骤主键 |
| data[].caseId | Integer | 用例 ID |
| data[].scriptNo | Integer | 关联脚本编号 |
| data[].stepDesc | String | 步骤描述 |
| data[].stepExpect | String | 预期结果 |
| data[].stepOrder | Integer | 排序号 |
| data[].scriptType | Integer | 脚本类型 |
| data[].parallelFlag | Integer | 并行标记 |
| data[].scriptStatus | Integer | 脚本状态 |
| data[].scriptName | String | 脚本名称（由脚本服务回填） |

### 实现意图

查询 `case_step` 表，再调用 `ScriptV3Api.getCaseStepScriptList` 批量获取脚本名称和状态填充到结果中。

### 调用链

```
CaseStepController.getCaseStepList
└─ CaseStepServiceImpl.getCaseStepDTOList
   ├─ ICaseStepDAO.selectListByCaseId (db_case.case_step)
   └─ ScriptV3Api.getCaseStepScriptList → [脚本服务](../../../脚本服务/00-首页.md)
```

### 涉及表

- `db_case.case_step`

---

## 2. POST /v3/test_case/case_step — 添加步骤

### 入口

`CaseStepController.addCaseStep(@RequestBody CaseStepRequestDTO caseStep)`（带 `@OperateLog(CASE_STEP_ADD)`）

### 请求参数（CaseStepRequestDTO，继承 CaseStep）

| 字段 | 必填 | 说明 |
|---|---|---|
| caseId | 是 | 用例 ID |
| stepDesc | 否 | 步骤描述（与 stepExpect/scriptNo 至少一个非空） |
| stepExpect | 否 | 预期结果（与 stepDesc/scriptNo 至少一个非空） |
| scriptNo | 否 | 关联脚本编号 |
| scriptType | 条件 | 传 scriptNo 时必填 |
| scriptStatus | 否 | 脚本状态 |
| userId | 否 | 操作人 |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 新步骤主键 ID。

### 实现意图

自动计算排序号 = 同级最大 + 1。若关联脚本且状态非"有效"（!=1），则将步骤脚本状态置为 0，同步更新用例 `case_check_status` 为无效，并通过 [实时任务服务](../../../任务管理服务/00-首页.md) 同步该状态。新增后更新用例 `update_time`。

### 关联横切

- `@OperateLog(operateLog = OperateLogEnum.CASE_STEP_ADD)`：AOP 写操作日志。

### 涉及表

- `db_case.case_step`
- `db_case.case_info`

---

## 3. PUT /v3/test_case/case_step — 更新步骤

### 入口

`CaseStepController.editCaseStep(@RequestBody CaseStepRequestDTO caseStep)`（带 `@OperateLog(CASE_STEP_UPDATE)`）

### 请求参数

同 `CaseStepRequestDTO`，`stepId` → `id` 映射。

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

### 实现意图

更新时不改动 `stepOrder`（排序通过移动接口独立操作）。更新后刷新用例 `update_time`。

### 涉及表

- `db_case.case_step`
- `db_case.case_info`

---

## 4. DELETE /v3/test_case/case_step/{step_id} — 删除步骤

### 入口

`CaseStepController.deleteCaseStep(@PathVariable Integer stepId, @RequestParam Integer caseId, Integer userId)`（带 `@OperateLog(CASE_STEP_DELETE)`）

### 请求参数

| 字段 | 必填 | 说明 |
|---|---|---|
| step_id | 是 | 步骤 ID |
| case_id | 是 | 用例 ID（双重校验） |
| user_id | 否 | 操作人 |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

### 实现意图

按 `stepId` + `caseId` 双重条件物理删除，防止误删其他用例的步骤。

### 涉及表

- `db_case.case_step`

---

## 5. POST /v3/test_case/case_step/move — 步骤移动

### 入口

`CaseStepController.moveCaseStep(@RequestBody CaseStepMoveDTO caseStepMoveDTO)`（带 `@OperateLog(CASE_STEP_MOVE)`）

### 请求参数（CaseStepMoveDTO）

| 字段 | 必填 | 说明 |
|---|---|---|
| caseId | 是 | 用例 ID |
| caseStepId | 是 | 步骤 ID |
| caseStepOrder | 是 | 当前排序号 |
| targetCaseStepOrder | 是 | 目标排序号 |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

### 实现意图

`@Transactional`，避免同位置移动。先调用 DAO 调整区间内其他步骤的排序号，再更新目标步骤的 `stepOrder`。

### 涉及表

- `db_case.case_step`

---

## 6. PUT /v3/test_case/case_step/case_steps — 批量更新步骤（online）

### 入口

`CaseStepController.updateCaseStep(@RequestParam Integer caseId, @RequestBody CaseStepsDTO caseStepsDTO)`（带 `@OperateLog(CASE_STEP_UPDATE_ONLINE)`）

### 请求参数（CaseStepsDTO）

| 字段 | 必填 | 说明 |
|---|---|---|
| userId | 否 | 操作人 |
| projectId | 是 | 项目 ID |
| caseSteps | 是 | 步骤列表 `List<CaseStep>`（全量覆盖） |

### 响应结构

`ResponseResult<BaseDataResultDTO>`。

### 实现意图

`@Transactional`，"先删后插"全量覆盖模式：①删除该用例所有旧步骤 ②按脚本类型（APP/WEB/PC）分别调用脚本服务查询审核状态 ③重新插入步骤（序号从 0 开始递增，`parallelFlag=0`）④汇总状态：任一步骤脚本无效则整体 `case_check_status=0`，同步到 `case_info` 并通过 [实时任务服务](../../../任务管理服务/00-首页.md) 同步。

### 调用链

```
CaseStepController.updateCaseStep
├─ @OperateLog(CASE_STEP_UPDATE_ONLINE) AOP
└─ CaseStepServiceImpl.updateCaseStepList (@Transactional)
   ├─ ICaseStepDAO.delete (db_case.case_step, 清空旧步骤)
   ├─ ScriptV3Api.getScriptList → [脚本服务](../../../脚本服务/00-首页.md) (按APP/WEB/PC分批查脚本状态)
   ├─ ICaseStepDAO.batchInsert (db_case.case_step)
   ├─ ICaseInfoDAO.updateById (db_case.case_info, case_check_status)
   └─ RealTaskV3Api.syncCaseIdStatus → [实时任务服务](../../../任务管理服务/00-首页.md)
```

### 关键代码摘录

```java
iCaseStepDAO.delete(new LambdaQueryWrapper<CaseStep>().eq(CaseStep::getCaseId, caseId));
for (CaseStep caseStep : caseSteps) {
    if (ScriptTypeEnum.APP.getType().equals(caseStep.getScriptType())) {
        appScriptNos.add(caseStep.getScriptNo());
    } // ... WEB, PC 同理
}
// 按脚本类型分别查询脚本审核状态
getScriptStatus(appScriptNos, projectId, ScriptTypeEnum.APP, scriptNoWithScriptStatus);
// 回填 scriptStatus 并写入
iCaseStepDAO.batchInsert(caseSteps);
```

### 涉及表

- `db_case.case_step`
- `db_case.case_info`

---

## 7. POST /v3/test_case/case_step/cancel_associated_script — 取消关联脚本

### 入口

`CaseStepController.cancelScript(@RequestBody CaseStepRequestDTO caseStep)`（带 `@OperateLog(CASE_STEP_UPDATE)`）

### 请求参数（CaseStepRequestDTO）

| 字段 | 必填 | 说明 |
|---|---|---|
| caseId | 是 | 用例 ID |
| stepId | 是 | 步骤 ID |

### 响应结构

`ResponseResult<BaseDataResultDTO>`。

### 实现意图

将该步骤的 `scriptNo` 和 `scriptType` 置为 null，保留描述和预期结果字段不变。更新后刷新用例 `update_time`。

```java
lambdaUpdateWrapper.set(CaseStep::getScriptNo, null);
lambdaUpdateWrapper.set(CaseStep::getScriptType, null);
```

### 涉及表

- `db_case.case_step`

