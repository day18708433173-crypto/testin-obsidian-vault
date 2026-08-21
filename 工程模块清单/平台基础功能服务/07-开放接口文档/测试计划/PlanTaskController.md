# PlanTaskController — 测试计划任务（计划-任务关联）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/plan/PlanTaskController.java`
> 类级路由：`/test_plan`
> Service 实现：`cn.testin.business.impl.plan.PlanTaskServiceImpl`（约 3200 行，本控制器全部委托给 `IPlanTaskService`）
> 业务：测试计划/子计划与任务模板的关联管理——添加、查询、排序、移动、删除，以及任务模板变更后向计划侧的数据同步。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 | 操作日志 |
|---|---|---|---|---|---|
| 1 | POST | `/v3/test_plan/plan_tasks` | addPlanTaskList | 给子计划下添加任务列表 | `RELATION_PLAN_TASK` |
| 2 | GET | `/v3/test_plan/plan_tasks` | selectPlanTasksByCondition | 查询任务列表（分页） | 无 |
| 3 | POST | `/v3/test_plan/plan_tasks/sub_plan_tree` | selectSubPlanTreeByCondition | 测试计划获取子计划的树结构 | 无 |
| 4 | PUT | `/v3/test_plan/plan_tasks/orders` | updatePlanTaskOrders | 更新任务列表的顺序 | 无 |
| 5 | PUT | `/v3/test_plan/plan_tasks/move/{plan_task_id}` | movePlanTaskOrders | 移动关联的任务 | 无 |
| 6 | POST | `/v3/test_plan/plan_tasks/relations` | selectTaskInfoRelations | 任务模板id是否关联了测试计划 | 无 |
| 7 | GET | `/v3/test_plan/task_infos` | selectTaskInfosByCondition | 查询可添加的任务信息（分页） | 无 |
| 8 | PUT | `/v3/test_plan/task_infos/{task_id}` | taskInfoUpdate | 任务模板更新后将相关数据同步到测试计划 | 无 |
| 9 | DELETE | `/v3/test_plan/plan_tasks/{plan_task_id}` | deletePlanTask | 删除一条关联的任务信息 | `PLAN_TASK_REMOVE` |
| 10 | POST | `/v3/test_plan/plan_tasks/batch_delete` | deletePlanTasks | 批量移除任务 | `PLAN_TASK_REMOVE` |
| 11 | POST | `/v3/test_plan/template_device/batch_update` | batchUpdateTemplateDevice | 任务模板批量更新（设备） | 无 |
| 12 | POST | `/v3/test_plan/template_device/device_type` | deviceType | 任务模板中获取有哪些设备类型 | 无 |
| 13 | GET | `/v3/test_plan/plan_detail_info` | getPlanTaskInfoAllInfo | 获取计划任务全部详情信息 | 无 |
| 14 | POST | `/v3/test_plan/update_template_content` | updateSourceTag | 批量更新模板数据源标签（任务树） | 无 |
| 15 | POST | `/v3/test_plan/update_sub_plan_template_content` | updateSubPlanTemplateContent | 批量更新模板数据源标签（子计划列表） | 无 |
| 16 | POST | `/v3/test_plan/plan_tasks/batch_delete_sub_plan` | deletePlanTasksSubPlan | 批量移除任务（子计划任务列表） | `PLAN_TASK_REMOVE` |
| 17 | POST | `/v3/test_plan/plan_tasks/batch_update_device_sub_plan` | batchUpdateDeviceSubPlan | 子计划列表批量修改设备 | 无 |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`；写操作大多返回 `BaseDataResultDTO { Long result }`（影响行数或主键）。
分页默认值：`page = Constants.PAGE_DEFAULT`，`pageSize = Constants.PAGE_SIZE_FIFTY`（50）。
GET 查询接口带 `@UnderlineToCamel`：下划线 query 参数自动转驼峰绑定 DTO。

---

## 1. POST /v3/test_plan/plan_tasks — 给子计划下添加任务列表

### 入口

`PlanTaskController.addPlanTaskList(@RequestBody @Valid PlanTaskRequestDTO request)`

### 请求参数（PlanTaskRequestDTO，JSON Body，@Valid）

以子计划为单位批量关联任务模板；具体字段见 `cn.testin.dto.request.plan.PlanTaskRequestDTO`。

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 新增关联记录数。

### 实现意图

将一批任务模板按顺序关联到指定子计划下，生成 plan_task 关联记录；Service 方法带 `@Transactional(rollbackFor = Exception.class)`，整体成败一致。

### 调用链

```
PlanTaskController.addPlanTaskList
├─ @OperateLog(RELATION_PLAN_TASK) AOP 记录操作日志
└─ PlanTaskServiceImpl.addPlanTasks (@Transactional)
```

### 关联横切

- `@OperateLog(operateLog = OperateLogEnum.RELATION_PLAN_TASK)`：AOP 写操作日志。

---

## 2. GET /v3/test_plan/plan_tasks — 查询任务列表

### 入口

`PlanTaskController.selectPlanTasksByCondition(@Valid PlanTaskConditionRequestDTO request)`（`@UnderlineToCamel`）

### 请求参数（PlanTaskConditionRequestDTO，Query）

| 字段 | 必填 | 说明 |
|---|---|---|
| page | 否 | 缺省 `Constants.PAGE_DEFAULT` |
| pageSize | 否 | 缺省 50（`PAGE_SIZE_FIFTY`） |
| 其余条件字段 | 否 | 见 `PlanTaskConditionRequestDTO`（下划线命名自动转驼峰） |

### 响应结构

`ResponseResult<BasePageListResponseDTO<PlanTaskDetailResponseDTO>>`：分页任务详情列表。

### 实现意图

按条件分页查询计划下已关联的任务详情；结果经 `dealDataSource` 等后处理补充数据源/设备信息。

### 调用链

```
PlanTaskController.selectPlanTasksByCondition
└─ PlanTaskServiceImpl.selectPlanTasksByCondition
   └─ dealDataSource(planTaskDetailList, type, eid, projectId)
```

---

## 3. POST /v3/test_plan/plan_tasks/sub_plan_tree — 子计划树结构

### 入口

`PlanTaskController.selectSubPlanTreeByCondition(@RequestBody SubPlanTreeConditionRequestDTO request)`

### 请求参数（SubPlanTreeConditionRequestDTO，JSON Body）

| 字段 | 必填 | 说明 |
|---|---|---|
| page | 否 | 缺省 `PAGE_DEFAULT` |
| pageSize | 否 | 缺省 50 |
| 其余条件字段 | 否 | 见 `SubPlanTreeConditionRequestDTO` |

### 响应结构

`ResponseResult<SubPlanPageListResponseDTO<SubPlanInfoTreeResponseDTO>>`：分页的子计划树。

### 调用链

```
PlanTaskController.selectSubPlanTreeByCondition
└─ PlanTaskServiceImpl.selectSubPlanTreeByCondition
```

---

## 4. PUT /v3/test_plan/plan_tasks/orders — 更新任务列表顺序

### 入口

`PlanTaskController.updatePlanTaskOrders(@RequestBody @Valid PlanTaskOrderListRequestDTO request)`

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 更新行数。

### 实现意图

按请求中的 id 列表批量重排 plan_task 的 order 字段；`@Transactional(rollbackFor = Exception.class)`。

### 调用链

```
PlanTaskController.updatePlanTaskOrders
└─ PlanTaskServiceImpl.updatePlanTaskOrders (@Transactional)
```

---

## 5. PUT /v3/test_plan/plan_tasks/move/{plan_task_id} — 移动关联的任务

### 入口

`PlanTaskController.movePlanTaskOrders(@PathVariable("plan_task_id") Long planTaskId, @RequestBody PlanTaskMoveRequestDTO request)`

Path 中的 `plan_task_id` 被塞回 `request.setId(planTaskId)` 后交 Service。

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

### 实现意图

将单条关联任务移动到目标位置（跨子计划或排序变更），`@Transactional`。

### 调用链

```
PlanTaskController.movePlanTaskOrders
└─ PlanTaskServiceImpl.movePlanTaskOrders (@Transactional)
```

---

## 6. POST /v3/test_plan/plan_tasks/relations — 任务模板是否已关联计划

### 入口

`PlanTaskController.selectTaskInfoRelations(@RequestBody TaskInfoRelationsRequestDTO request)`

### 请求参数（TaskInfoRelationsRequestDTO，JSON Body）

| 字段 | 必填 | 说明 |
|---|---|---|
| taskIds | 是 | 任务模板 id 列表；为空抛 `GeneralException(paraInvalid, 查询的任务列表不能为空)` |

### 响应结构

`ResponseResult<BaseListResponseDTO<TaskInfoWithPlanInfoResponseDTO>>`：每个任务模板及其已关联的计划信息。

### 调用链

```
PlanTaskController.selectTaskInfoRelations
├─ 校验 taskIds 非空（Controller 层）
└─ PlanTaskServiceImpl.selectTaskInfoRelations
```

---

## 7. GET /v3/test_plan/task_infos — 查询可添加的任务信息

### 入口

`PlanTaskController.selectTaskInfosByCondition(@Valid PlanTaskConditionRequestDTO request)`（`@UnderlineToCamel`）

### 请求参数

| 字段 | 必填 | 说明 |
|---|---|---|
| subPlanInfoId | 是 | 子计划 id；为 null 抛 `GeneralException(paraInvalid, 子计划不能为空)` |
| page / pageSize | 否 | 缺省 PAGE_DEFAULT / 50 |

### 响应结构

`ResponseResult<BasePageListResponseDTO<PlanTaskDetailResponseDTO>>`。

### 调用链

```
PlanTaskController.selectTaskInfosByCondition
├─ 校验 subPlanInfoId 非空（Controller 层）
└─ PlanTaskServiceImpl.selectTaskInfosByCondition
```

---

## 8. PUT /v3/test_plan/task_infos/{task_id} — 模板更新同步到计划

### 入口

`PlanTaskController.taskInfoUpdate(@PathVariable("task_id") Integer taskId, @RequestBody TaskInfoUpdateRequestDTO request)`

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 同步更新的行数。

### 实现意图

任务模板（task_info）内容更新后，把变更同步到已引用该模板的测试计划侧数据（plan_task 及关联明细）。

### 调用链

```
PlanTaskController.taskInfoUpdate
└─ PlanTaskServiceImpl.updateByTaskId(taskId, request)
```

---

## 9. DELETE /v3/test_plan/plan_tasks/{plan_task_id} — 删除单条关联任务

### 入口

`PlanTaskController.deletePlanTask(@PathVariable("plan_task_id") Long planTaskId, @RequestParam("user_id") Integer userId)`

### 请求参数

| 字段 | 来源 | 必填 | 说明 |
|---|---|---|---|
| plan_task_id | Path | 是 | 关联记录主键 |
| user_id | Query | 是 | 操作人（写入 update_user_id/删除标记） |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

### 调用链

```
PlanTaskController.deletePlanTask
├─ @OperateLog(PLAN_TASK_REMOVE)
└─ PlanTaskServiceImpl.deleteById (@Transactional)
```

---

## 10. POST /v3/test_plan/plan_tasks/batch_delete — 批量移除任务

### 入口

`PlanTaskController.deletePlanTasks(@RequestBody PlanTaskDeleteRequestDTO request)`

### 请求参数（PlanTaskDeleteRequestDTO，JSON Body）

| 字段 | 必填 | 说明 |
|---|---|---|
| condition | 二选一 | 按条件删除 |
| planTaskIds | 二选一 | 按 id 列表删除；两者都为空抛 `GeneralException(paraInvalid, 删除数据不能为空)` |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 删除行数。

### 调用链

```
PlanTaskController.deletePlanTasks
├─ 校验 condition/planTaskIds 至少其一（Controller 层）
├─ @OperateLog(PLAN_TASK_REMOVE)
└─ PlanTaskServiceImpl.deletePlanTasks (@Transactional)
```

---

## 11. POST /v3/test_plan/template_device/batch_update — 任务模板批量更新设备

### 入口

`PlanTaskController.batchUpdateTemplateDevice(@RequestBody TemplateBatchUpdateDeviceDTO request)`

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 更新行数。

### 调用链

```
PlanTaskController.batchUpdateTemplateDevice
└─ PlanTaskServiceImpl.batchUpdateTemplateDevice
```

---

## 12. POST /v3/test_plan/template_device/device_type — 模板包含的设备类型

### 入口

`PlanTaskController.deviceType(@RequestBody TemplateBatchUpdateDeviceDTO request)`

### 响应结构

`ResponseResult<CaseDeviceTypeResponseDTO>`：模板任务覆盖到的设备类型集合。

### 调用链

```
PlanTaskController.deviceType
└─ PlanTaskServiceImpl.templateHaveType
```

---

## 13. GET /v3/test_plan/plan_detail_info — 计划任务全部详情

### 入口

`PlanTaskController.getPlanTaskInfoAllInfo(@RequestParam("plan_info_id") Long planInfoId)`

### 请求参数

| 字段 | 来源 | 必填 | 说明 |
|---|---|---|---|
| plan_info_id | Query | 是 | 测试计划 id |

### 响应结构

`ResponseResult<PlanInfoAllDetailInfoResponseDTO>`：计划 + 子计划 + 任务的聚合详情。

### 调用链

```
PlanTaskController.getPlanTaskInfoAllInfo
└─ PlanTaskServiceImpl.getPlanTaskInfoAllInfo
```

---

## 14. POST /v3/test_plan/update_template_content — 批量更新模板数据源标签（任务树）

### 入口

`PlanTaskController.updateSourceTag(@RequestBody TemplateContentRequestDTO requestDTO)`

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 更新数量（Long）。

### 调用链

```
PlanTaskController.updateSourceTag
└─ PlanTaskServiceImpl.updateTemplateContent
```

---

## 15. POST /v3/test_plan/update_sub_plan_template_content — 批量更新模板数据源标签（子计划列表）

### 入口

`PlanTaskController.updateSubPlanTemplateContent(@RequestBody SubPlanTemplateContentUpdateRequestDTO requestDTO)`

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 更新数量（Long）。

### 调用链

```
PlanTaskController.updateSubPlanTemplateContent
└─ PlanTaskServiceImpl.subPlanUpdateTemplateContent
```

---

## 16. POST /v3/test_plan/plan_tasks/batch_delete_sub_plan — 批量移除任务（子计划任务列表）

### 入口

`PlanTaskController.deletePlanTasksSubPlan(@RequestBody SubPlanTaskDeleteRequestDTO request)`

### 请求参数（SubPlanTaskDeleteRequestDTO，JSON Body）

| 字段 | 必填 | 说明 |
|---|---|---|
| condition | 二选一 | 按条件删除 |
| planTaskIds | 二选一 | 按 id 列表删除；两者都为空抛 `GeneralException(paraInvalid, 删除数据不能为空)` |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 删除行数。

### 调用链

```
PlanTaskController.deletePlanTasksSubPlan
├─ 校验 condition/planTaskIds 至少其一（Controller 层）
├─ @OperateLog(PLAN_TASK_REMOVE)
└─ PlanTaskServiceImpl.deleteSubPlanTasks (@Transactional)
```

---

## 17. POST /v3/test_plan/plan_tasks/batch_update_device_sub_plan — 子计划列表批量修改设备

### 入口

`PlanTaskController.batchUpdateDeviceSubPlan(@RequestBody SubPlanTaskTemplateDeviceRequestDTO request)`

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 更新行数。

### 调用链

```
PlanTaskController.batchUpdateDeviceSubPlan
└─ PlanTaskServiceImpl.batchUpdateSubPlanTaskTemplateDevice
```

---

## 备注

- 本 Controller 为纯转发层：参数兜底校验（taskIds / subPlanInfoId / 删除条件非空）在 Controller 完成，业务与事务均在 `PlanTaskServiceImpl`。
- 事务方法（已核实 `@Transactional(rollbackFor = Exception.class)`）：`addPlanTasks`、`updatePlanTaskOrders`、`movePlanTaskOrders`、`deleteById`、`deletePlanTasks`、`deleteSubPlanTasks`。
- Service 内部复用方法（不对应 HTTP 端点）：`getTaskCountByPlanInfoIds`、`getTaskCountWithSubPlanInfoIds`、`getTaskCountWithRelationType`、`deleteByPlanInfoId`、`deleteBySubPlanInfoId`、`selectPlanTasksByConditionNoPage` 等，供计划删除级联、统计与内部查询使用。
- 请求 DTO 统一在 `cn.testin.dto.request.plan`，响应 DTO 在 `cn.testin.dto.response.plan`（设备类型响应在 `dto.response.testCase.CaseDeviceTypeResponseDTO`）。

相关文档：[00-分支索引](00-分支索引.md) · [PlanInfoConfigController](PlanInfoConfigController.md) · [PlanInfoExecutePeriodController](PlanInfoExecutePeriodController.md) · [PlanInfoScheduledController](PlanInfoScheduledController.md)
