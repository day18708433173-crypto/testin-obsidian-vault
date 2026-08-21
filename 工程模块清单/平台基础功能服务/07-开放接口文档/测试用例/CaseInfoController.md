# CaseInfoController — 测试用例信息

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/testCase/CaseInfoController.java`
> 类级路由：`/v3/test_case/case_info`
> Service 实现：`cn.testin.business.impl.testCase.CaseInfoServiceImpl`（约 860 行，委托给 `ICaseInfoService`）
> 业务：测试用例的增删改查、目录移动、数据源关联/解除、脚本转用例、脚本状态同步。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 | 操作日志 |
|---|---|---|---|---|---|
| 1 | GET | `/v3/test_case/case_info` | getCaseInfo | 根据 caseId 获取用例详情 | 无 |
| 2 | POST | `/v3/test_case/case_info/case_infos` | getCaseInfoList | 分页查询用例列表（含标签、数据源、执行次数） | 无 |
| 3 | POST | `/v3/test_case/case_info/case_base_infos` | getCaseBaseInfo | 分页查询用例基本信息（含标签、脚本类型） | 无 |
| 4 | POST | `/v3/test_case/case_info` | addTestCase | 新增用例（含步骤、标签） | `CASE_ADD` |
| 5 | PUT | `/v3/test_case/case_info` | updateTestCase | 修改用例信息 | `CASE_UPDATE` |
| 6 | DELETE | `/v3/test_case/case_info/{case_id}` | deleteTestCase | 删除用例（逻辑删除） | `CASE_DELETE` |
| 7 | POST | `/v3/test_case/case_info/move` | moveTestCase | 移动用例到指定目录 | 无 |
| 8 | GET | `/v3/test_case/case_info/case_infos` | getCaseInfoList | 根据数据源 ID 获取关联的用例列表 | 无 |
| 9 | POST | `/v3/test_case/case_info/source/add_case_relation` | addCaseRelation | 用例关联数据源 | `CASE_RELATION_DATA_SOURCE` |
| 10 | POST | `/v3/test_case/case_info/source/remove_case_relation` | removeCaseRelation | 解除用例与数据源的关联 | `CASE_REMOVE_DATA_SOURCE` |
| 11 | GET | `/v3/test_case/case_info/source` | getCaseSourceRelation | 获取用例关联的数据源信息 | 无 |
| 12 | POST | `/v3/test_case/case_info/case_infos/case_steps` | getCaseInfoAndStepDTO | 批量查询用例及其步骤信息 | 无 |
| 13 | POST | `/v3/test_case/case_info/case_infos/script_transform_case` | scriptTransformCase | 脚本转换成用例（含目录/步骤自动创建） | 无 |
| 14 | POST | `/v3/test_case/case_info/update_case_version` | batchUpdataCaseVersion | 批量更新用例版本号 | 无 |
| 15 | POST | `/v3/test_case/case_info/sync_script_no_status` | syncScriptNoStatus | 同步脚本审核状态到用例步骤 | 无 |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`；写操作大多返回 `BaseDataResultDTO { Long result }`（影响行数或主键）。
分页默认值：`page = Constants.PAGE_DEFAULT`，`pageSize = Constants.PAGE_SIZE_DEFAULT`。

---

## 1. GET /v3/test_case/case_info — 获取用例详情

### 入口

`CaseInfoController.getCaseInfo(@RequestParam Integer caseId, Integer eid, Integer projectId)`

### 请求参数（Query）

| 字段 | 必填 | 说明 |
|---|---|---|
| case_id | 是 | 用例 ID |
| eid | 否 | 企业 ID |
| project_id | 否 | 项目 ID |

### 响应结构

`ResponseResult<CaseInfoDTO>`，`data` 为 `CaseInfoDTO`（继承 `CaseInfo`，额外包含 `caseTagList` 标签列表）。

### 返回参数（CaseInfoDTO 元素结构，继承 CaseInfo）

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 用例对象 |
| data.id | Integer | 用例主键 |
| data.eid | Integer | 企业 ID |
| data.projectId | Integer | 项目 ID |
| data.caseUUID | String | 用例 UUID |
| data.caseName | String | 用例名称 |
| data.caseLevel | Integer | 用例等级 |
| data.casePurpose | String | 测试目的 |
| data.caseVersion | String | 版本号 |
| data.caseDirId | Integer | 目录 ID |
| data.caseRemark | String | 备注 |
| data.caseStatus | Integer | 用例状态 |
| data.caseCheckStatus | Integer | 审核状态 |
| data.createUserId | Integer | 创建人 ID |
| data.updateUserId | Integer | 更新人 ID |
| data.status | Integer | 记录状态（ENABLED/DISABLED） |
| data.createTime | Date | 创建时间 |
| data.updateTime | Date | 更新时间 |
| data.caseId | Integer | 用例 ID（DTO 冗余字段） |
| data.dataSourceId | Long | 关联数据源 ID |
| data.dataSourceName | String | 关联数据源名称 |
| data.userId | Integer | 用户 ID |
| data.userName | String | 用户名 |
| data.caseTagList | Array\<String\> | 标签列表 |
| data.caseStepList | Array\<CaseStep\> | 步骤列表（字段见 CaseStepController） |
| data.executeNum | Integer | 执行次数 |
| data.scriptTypes | Array\<Integer\> | 关联脚本类型列表 |

### 实现意图

按 caseId（可叠加 eid/projectId 做租户过滤）查询单条用例，自动附带标签信息。查不到时返回空 `CaseInfoDTO`。

### 调用链

```
CaseInfoController.getCaseInfo
└─ CaseInfoServiceImpl.getCaseInfoById
   ├─ ICaseInfoDAO.selectOne (db_case.case_info)
   └─ CaseTagServiceImpl.getCaseTagListByCaseId (标签)
```

### 涉及表

- `db_case.case_info`

---

## 2. POST /v3/test_case/case_info/case_infos — 分页查询用例列表

### 入口

`CaseInfoController.getCaseInfoList(@RequestBody CaseInfoConditionRequestDTO request)`（`@UnderlineToCamel`）

### 请求参数（CaseInfoConditionRequestDTO，JSON Body）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 否 | 企业 ID |
| projectId | 否 | 项目 ID |
| page | 否 | 页码，缺省 `PAGE_DEFAULT` |
| pageSize | 否 | 每页条数，缺省 `PAGE_SIZE_DEFAULT` |
| caseName | 否 | 用例名称（模糊） |
| caseLevel | 否 | 用例等级 |
| caseTagList | 否 | 标签列表（交集查询） |
| caseStatus | 否 | 用例状态 |
| caseDirId | 否 | 目录 ID（会递归展开所有子目录） |
| caseCreateUserName | 否 | 创建人用户名 |
| caseUpdateUserName | 否 | 更新人用户名 |
| caseIdList | 否 | 用例 ID 列表 |

### 响应结构

`ResponseResult<BasePageListResponseDTO<CaseInfoDTO>>`，每条 `CaseInfoDTO` 额外填充：`caseTagList`（标签）、`executeNum`（执行次数）、`dataSourceId/dataSourceName`（关联数据源）。

### 实现意图

多条件组合分页查询。关键处理：①递归展开目录为所有子目录 ID ②用户名转用户 ID 列表 ③标签名转用例 ID 列表（交集） ④批量查执行记录次数 ⑤批量查数据源关联。

### 调用链

```
CaseInfoController.getCaseInfoList
└─ CaseInfoServiceImpl.getCaseInfoListByCondition
   ├─ CaseDirServiceImpl.getAllChildCaseDir (递归展开子目录)
   ├─ UserService.getUserIdByUserName (用户名→ID)
   ├─ CaseTagServiceImpl.getCaseIdListByTag (标签→用例ID)
   ├─ ICaseInfoDAO.countByCondition / getCaseInfoLByCondition (db_case.case_info)
   ├─ ReportCaseV3Api.getReportCaseInfo → 报告用例服务
   ├─ DataSourceV3Api.getCaseDataSourceByCondition → [数据源服务](../../../数据源/00-首页.md)
   └─ CaseTagServiceImpl.getCaseTagList (标签)
```

### 涉及表

- `db_case.case_info`
- `db_case.case_dir`

---

## 3. POST /v3/test_case/case_info/case_base_infos — 查询用例基本信息

### 入口

`CaseInfoController.getCaseBaseInfo(@RequestBody CaseInfoConditionRequestDTO request)`

### 请求参数

同 `CaseInfoConditionRequestDTO`，额外可用 `needTagInfo` 控制是否返回标签，`caseIdList` 必填。

### 响应结构

`ResponseResult<BasePageListResponseDTO<CaseInfoDTO>>`，每条额外填充：`caseTagList`（可选）、`scriptTypes`（该用例关联的脚本类型列表）。

### 实现意图

轻量版分页查询，用于列表场景。按 `caseIdList` 查询，额外计算每个用例涉及的脚本类型集合。

### 涉及表

- `db_case.case_info`
- `db_case.case_step`

---

## 4. POST /v3/test_case/case_info — 新增用例

### 入口

`CaseInfoController.addTestCase(@RequestBody CaseInfoDTO CaseInfoDTO)`（带 `@OperateLog(CASE_ADD)`）

### 请求参数（CaseInfoDTO，JSON Body）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 是 | 企业 ID |
| projectId | 是 | 项目 ID |
| caseName | 是 | 用例名称 |
| caseDirId | 是 | 目录 ID |
| caseLevel | 否 | 等级，缺省 0 |
| casePurpose | 否 | 测试目的 |
| caseVersion | 否 | 版本号 |
| caseRemark | 否 | 备注 |
| caseStatus | 否 | 用例状态，缺省"待评审" |
| userId | 是 | 创建人/更新人 ID |
| caseTagList | 否 | 标签列表 |
| caseStepList | 否 | 步骤列表（`List<CaseStep>`） |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 新增用例主键 ID。

### 实现意图

`@Transactional` 批量操作：①生成 `caseUUID` ②插入 `case_info` ③批量插入 `case_step`（检测脚本状态，有无效脚本则标记用例状态为无效）④同步标签 ⑤清除目录缓存。若步骤含无效脚本会通过 [实时任务服务](../../../任务管理服务/00-首页.md) 同步用例审核状态。

### 调用链

```
CaseInfoController.addTestCase
├─ @OperateLog(CASE_ADD) AOP
└─ CaseInfoServiceImpl.addCaseInfo (@Transactional)
   ├─ generateCaseUUID → UUID.randomUUID
   ├─ ICaseInfoDAO.insert (db_case.case_info)
   ├─ ICaseStepDAO.batchInsert (db_case.case_step)
   ├─ CaseTagServiceImpl.updateCaseTagList (db_case.case_tag)
   ├─ CaseDirServiceImpl.updateCaseNum (清除Redis目录用例数缓存)
   └─ RealTaskV3Api.syncCaseIdStatus → [实时任务服务](../../../任务管理服务/00-首页.md)
```

### 关键代码摘录

```java
caseInfoDTO.setCaseUUID(generateCaseUUID());
caseInfoDTO.setStatus(CaseInfoStatusEnum.ENABLED.getCode());
caseInfoDTO.setCaseCheckStatus(CaseCheckStatusEnum.VALID.getCode());
iCaseInfoDAO.insert(caseInfoDTO);
if (!CollectionUtils.isEmpty(caseInfoDTO.getCaseStepList())) {
    for (int i = 0; i < caseInfoDTO.getCaseStepList().size(); i++) {
        CaseStep caseStep = caseInfoDTO.getCaseStepList().get(i);
        if (caseStep.getScriptStatus() != null && !caseStep.getScriptStatus().equals(1)) {
            caseStep.setScriptStatus(0);
            valid = CaseCheckStatusEnum.INVALID.getCode();
        }
        caseStep.setCaseId(caseInfoDTO.getId());
        caseStep.setStepOrder(i);
    }
    iCaseStepDAO.batchInsert(caseInfoDTO.getCaseStepList());
}
```

### 涉及表

- `db_case.case_info`
- `db_case.case_step`
- `db_case.case_tag`

---

## 5. PUT /v3/test_case/case_info — 修改用例信息

### 入口

`CaseInfoController.updateTestCase(@RequestBody CaseInfoDTO CaseInfoDTO)`（带 `@OperateLog(CASE_UPDATE)`）

### 请求参数

同 `CaseInfoDTO`，需传 `caseId`（映射为 `id`）。

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

### 实现意图

更新用例基本信息 + 标签。更新时不改 `caseDirId`（目录归属独立通过移动接口变更）。

### 调用链

```
CaseInfoController.updateTestCase
├─ @OperateLog(CASE_UPDATE) AOP
└─ CaseInfoServiceImpl.updateCaseInfo (@Transactional)
   ├─ CaseTagServiceImpl.updateCaseTagList (标签同步)
   └─ ICaseInfoDAO.updateById (db_case.case_info)
```

### 涉及表

- `db_case.case_info`
- `db_case.case_tag`

---

## 6. DELETE /v3/test_case/case_info/{case_id} — 删除用例

### 入口

`CaseInfoController.deleteTestCase(@PathVariable Integer caseId, @RequestParam Integer userId)`（带 `@OperateLog(CASE_DELETE)`）

### 请求参数

| 字段 | 必填 | 说明 |
|---|---|---|
| case_id | 是 | 路径参数，用例 ID |
| user_id | 是 | 操作人 ID |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

### 实现意图

逻辑删除（`status → DISABLED`），附带：①调用 `DataSourceV3Api.removeCaseDataSourceRelation` 解除数据源关联 ②清除原目录用例数缓存 ③通过 [实时任务服务](../../../任务管理服务/00-首页.md) 同步用例状态。

### 调用链

```
CaseInfoController.deleteTestCase
├─ @OperateLog(CASE_DELETE) AOP
└─ CaseInfoServiceImpl.deleteCaseInfo
   ├─ ICaseInfoDAO.updateById (db_case.case_info, status→DISABLED)
   ├─ DataSourceV3Api.removeCaseDataSourceRelation → [数据源服务](../../../数据源/00-首页.md)
   ├─ CaseDirServiceImpl.updateCaseNum (清除Redis)
   └─ RealTaskV3Api.syncCaseIdStatus → [实时任务服务](../../../任务管理服务/00-首页.md)
```

### 涉及表

- `db_case.case_info`

---

## 7. POST /v3/test_case/case_info/move — 移动用例

### 入口

`CaseInfoController.moveTestCase(@RequestBody CaseInfoMoveDTO caseInfoMoveDTO)`

### 请求参数（CaseInfoMoveDTO）

| 字段 | 必填 | 说明 |
|---|---|---|
| caseId | 是 | 用例 ID |
| caseDirId | 是 | 目标目录 ID |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 影响行数。

### 实现意图

更新 `case_info.case_dir_id` 为目标目录，同时清除原目录和新目录的用例数 Redis 缓存。

### 涉及表

- `db_case.case_info`
- `db_case.case_dir`

---

## 8. GET /v3/test_case/case_info/case_infos?case_source_id= — 按数据源查用例

### 入口

`CaseInfoController.getCaseInfoList(@RequestParam Integer caseSourceId)`

### 请求参数（Query）

| 字段 | 必填 | 说明 |
|---|---|---|
| case_source_id | 是 | 数据源 ID |

### 响应结构

`ResponseResult<List<CaseInfoDTO>>`

### 实现意图

调用 `DataSourceV3Api.getCaseSourceRelation` 获取数据源关联的用例 ID 列表，再批量查询用例信息。

### 涉及表

- `db_case.case_info`

### 跨服务调用

- `DataSourceV3Api.getCaseSourceRelation` → [数据源服务](../../../数据源/00-首页.md)

---

## 9. POST /v3/test_case/case_info/source/add_case_relation — 关联数据源

### 入口

`CaseInfoController.addCaseRelation(@Valid @RequestBody CaseSourceRelationRequestDTO request)`（带 `@OperateLog(CASE_RELATION_DATA_SOURCE)`）

### 请求参数（CaseSourceRelationRequestDTO，@Valid）

| 字段 | 必填 | 说明 |
|---|---|---|
| caseSourceId | 是 | 数据源 ID |
| caseId | 是 | 用例 ID |
| projectId | 是 | 项目 ID |
| eid | 是 | 企业 ID |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 1L（成功）。

### 实现意图

校验用例存在后调用数据源服务建立关联，更新用例 `update_time`。

### 涉及表

- `db_case.case_info`

### 跨服务

- `DataSourceV3Api.addCaseDataSourceRelation` → [数据源服务](../../../数据源/00-首页.md)

---

## 10. POST /v3/test_case/case_info/source/remove_case_relation — 解除数据源关联

### 入口

`CaseInfoController.removeCaseRelation(@RequestBody CaseSourceRelationRequestDTO request)`（带 `@OperateLog(CASE_REMOVE_DATA_SOURCE)`）

### 请求参数

同 `CaseSourceRelationRequestDTO`。

### 响应结构

`ResponseResult<BaseDataResultDTO>`。

### 实现意图

校验用例存在后调用数据源服务解除关联。

### 涉及表

- `db_case.case_info`

### 跨服务

- `DataSourceV3Api.removeCaseDataSourceRelation` → [数据源服务](../../../数据源/00-首页.md)

---

## 11. GET /v3/test_case/case_info/source — 获取用例关联数据源

### 入口

`CaseInfoController.getCaseSourceRelation(@RequestParam Integer caseId)`

### 响应结构

`ResponseResult<CaseSource>`（含数据源 ID、名称等）。

### 实现意图

调用数据源服务获取该用例关联的数据源详情。

### 跨服务

- `DataSourceV3Api.getCaseDataSourceByCaseId` → [数据源服务](../../../数据源/00-首页.md)

---

## 12. POST /v3/test_case/case_info/case_infos/case_steps — 查用例及步骤

### 入口

`CaseInfoController.getCaseInfoAndStepDTO(@RequestBody CaseSourceRelationRequestDTO request)`

### 请求参数

| 字段 | 必填 | 说明 |
|---|---|---|
| caseIdList | 是 | 用例 ID 列表 |

### 响应结构

`ResponseResult<List<CaseInfoDTO>>`，每个 `CaseInfoDTO` 附带 `caseStepList`（步骤列表，按 `step_order` 升序）。

### 涉及表

- `db_case.case_info`
- `db_case.case_step`

---

## 13. POST /v3/test_case/case_info/case_infos/script_transform_case — 脚本转用例

### 入口

`CaseInfoController.scriptTransformCase(@RequestBody ScriptTransformCaseRequestDTO request)`

### 请求参数（ScriptTransformCaseRequestDTO）

| 字段 | 必填 | 说明 |
|---|---|---|
| scriptNoList | 否 | 指定脚本编号列表 |
| scriptDirId | 否 | 脚本目录 ID（批量转换整个目录） |
| projectId | 是 | 项目 ID |
| eid | 是 | 企业 ID |

### 响应结构

`ResponseResult<List<ScriptTransformCaseStepsResponseDTO>>`，每个元素包含转换结果，失败项排在前面。

### 实现意图

`@Transactional` 复杂编排：①调用脚本服务获取待转换脚本 ②分页（每批 20）处理 ③按脚本目录层级在 `db_case.case_dir` 创建对应目录树 ④创建用例（`case_info`）和步骤（`case_step`），同名已存在则跳过 ⑤清除目录用例数缓存。

### 调用链

```
CaseInfoController.scriptTransformCase
└─ CaseInfoServiceImpl.scriptTransformCase (@Transactional)
   ├─ ScriptV3Api.scriptTransformCaseScriptNos → [脚本服务](../../../脚本服务/00-首页.md)
   ├─ ScriptV3Api.scriptTransformCaseSteps → [脚本服务](../../../脚本服务/00-首页.md)
   ├─ ScriptV3Api.scriptTransformCaseDirs → [脚本服务](../../../脚本服务/00-首页.md)
   ├─ ICaseDirDAO (目录创建/查询 db_case.case_dir)
   ├─ ICaseInfoDAO.insert (db_case.case_info)
   └─ ICaseStepDAO.batchInsert (db_case.case_step)
```

### 涉及表

- `db_case.case_info`
- `db_case.case_step`
- `db_case.case_dir`

---

## 14. POST /v3/test_case/case_info/update_case_version — 批量更新版本

### 入口

`CaseInfoController.batchUpdataCaseVersion(@RequestBody CaseUpdateRequestDTO caseUpdateRequestDTO)`

### 请求参数（CaseUpdateRequestDTO）

| 字段 | 必填 | 说明 |
|---|---|---|
| newCaseVersion | 是 | 新版本号 |
| caseIds | 否 | 指定用例 ID 列表（与 condition 二选一） |
| condition | 否 | `CaseInfoConditionRequestDTO` 查询条件（全选模式） |
| projectId | 是 | 项目 ID |
| userId | 是 | 操作人 ID |

### 响应结构

`ResponseResult<BaseDataResultDTO>`，`data.result` = 更新行数。

### 实现意图

`@Transactional`。若未传 `caseIds` 则按 `condition` 全选查询用例 ID 列表（支持目录递归、用户名、标签过滤），再批量更新 `case_version`。

### 涉及表

- `db_case.case_info`

---

## 15. POST /v3/test_case/case_info/sync_script_no_status — 同步脚本状态

### 入口

`CaseInfoController.syncScriptNoStatus(@RequestBody ScriptCheckStatusSyncRequest request)`

### 请求参数（ScriptCheckStatusSyncRequest）

| 字段 | 必填 | 说明 |
|---|---|---|
| scriptNo | 是 | 脚本编号 |
| checkStatus | 是 | 审核状态 |
| scriptId | 否 | 脚本 ID |
| syncCount | 否 | 同步次数 |

### 响应结构

`ResponseResult<BaseDataResultDTO>`。

### 实现意图

加分布式锁（`LockKeyword.SCRIPT_NO_SYNC`），将脚本审核状态写入 `db_case.case_script_no_sync`，若状态变更则异步触发 `computeCaseScriptNoSync` 同步到关联用例步骤。

### 调用链

```
CaseInfoController.syncScriptNoStatus
└─ CaseInfoServiceImpl.syncScriptNoStatus (@DS(DB_PLAN), @Transactional)
   ├─ LockUtil.getLock (SCRIPT_NO_SYNC)
   ├─ ICaseScriptNoSyncDAO (db_case.case_script_no_sync)
   └─ CompletableFuture → ICaseScriptNoSyncService.computeCaseScriptNoSync
```

### 涉及表

- `db_case.case_script_no_sync`

