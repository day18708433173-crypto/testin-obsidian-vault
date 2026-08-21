# TestTemplateController — 任务模板 CRUD

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/mvc/controller/TestTemplateController.java`
> 类级路由：`/realweb/template`
> Service 实现：`TemplateService`（继承 MyBatis-Plus `ServiceImpl<QuartzJobMapper, QuartzJob>`）、`QuartzJobServiceImpl`

## 接口列表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|---|---|---|---|
| 1 | POST | `/v3/realweb/template/templates` | listByCondition | 条件分页查询模板列表 |
| 2 | DELETE | `/v3/realweb/template/batch_delete` | batchRemove | 批量删除模板 |
| 3 | POST | `/v3/realweb/template/save_json` | saveTemplate | 保存新模板 |
| 4 | GET | `/v3/realweb/template/copy` | copyTemplate | 复制模板 |
| 5 | POST | `/v3/realweb/template/update_template_content` | updateSourceTag | 更新模板内容（标签） |
| 6 | GET | `/v3/realweb/template/list_template_id` | getQuartzJobId | 查询模板ID列表 |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`。

---

## 1. POST /v3/realweb/template/templates — 条件分页查询模板列表

### 入口

`TestTemplateController.listByCondition(@RequestBody @Valid TemplateRequestDTO templateRequestDTO)`

### 请求参数（TemplateRequestDTO，JSON Body，@Valid 验证）

继承 `PageRequestDTO`（`@NotNull Integer projectId`, `Integer page`, `Integer pageSize`）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectId | Integer | 是 | 项目ID（@NotNull） |
| page | Integer | 否 | 页码 |
| pageSize | Integer | 否 | 每页大小 |
| taskName | String | 否 | 任务名称模糊搜索 |
| userIds | List&lt;Integer&gt; | 否 | 创建人过滤 |
| createStartTime | Long | 否 | 创建时间范围（起） |
| createEndTime | Long | 否 | 创建时间范围（止） |
| filterIds | List&lt;Integer&gt; | 否 | 要过滤掉的模板ID |
| ids | List&lt;Integer&gt; | 否 | 精确ID查询 |
| plan | boolean | 否 | 是否查询计划关联模板 |
| needContent | boolean | 否 | 是否需要返回完整 content JSON |
| needScriptDetail | boolean | 否 | 是否需要脚本详情 |
| needScriptAndDeviceBashInfo | boolean | 否 | 是否需要脚本/设备汇总信息 |
| checkStatus | boolean | 否 | 是否需要检查模板状态 |
| businessType | Integer | 否 | 业务类型过滤 |
| needDataSourceDetail | boolean | 否 | 是否需要数据源详情 |

### 响应结构

`ResponseResult<PageResponseDTO<TemplateResponseDTO>>`。

### 实现意图

查询 `quartz_job` 表（`deleteStatus=0`），按条件过滤（projectId、businessType、jobType=TEMPLATE、jobRule is null for plans、id列表等）。非 plan 查询时调用 [TestPlanV3Api.taskRelation](../其他ApiServlet/service-TestPlanV3Api.md) 获取模板与测试计划关联状态。根据 needContent/needScriptDetail 等 flag 从 `job_content` JSON 中扩展脚本数、设备数、脚本编号、设备详情、数据源等信息。

### 调用链

```
TestTemplateController.listByCondition
└─ TemplateService.listByCondition(templateRequestDTO)
   ├─ QuartzJobMapper.selectPage → MySQL quartz_job
   ├─ TestPlanV3Api.taskRelation(taskIds) → RealLogfile POST /v3/test_plan/plan_tasks/relations
   └─ 从 job_content JSON 解析扩展字段
```

---

## 2. DELETE /v3/realweb/template/batch_delete — 批量删除模板

### 入口

`TestTemplateController.batchRemove(@RequestBody TemplateRemoveRequestDTO requestDTO)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| templateIds | List&lt;Integer&gt; | 是 | 模板ID列表 |

### 响应结构

`ResponseResult<Integer>`，data = 影响行数。

### 实现意图

先调用 `TestPlanV3Api.taskRelation` 检查模板是否被测试计划关联。有关联则抛 `GeneralException("模板已被关联，无法删除")`。否则软删除（设置 `delete_status = 1`）。

### 调用链

```
TestTemplateController.batchRemove
└─ TemplateService.batchRemove(templateIds)
   ├─ TestPlanV3Api.taskRelation(templateIds) → RealLogfile
   └─ QuartzJobMapper.update → MySQL quartz_job (delete_status=1)
```

---

## 3. POST /v3/realweb/template/save_json — 保存新模板

### 入口

`TestTemplateController.saveTemplate(@RequestBody String jsonStr)`（原始 JSON 字符串）

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| (raw JSON) | String | 是 | 任务模板完整 JSON，其中 projectid 为必填 |

### 响应结构

`ResponseResult<Integer>`，data = 新 jobId。

### 实现意图

解析原始 JSON，`@Transactional` 保证一致性。执行 `handlerScriptAndeDevice` 计算 deviceTotal/deviceIds/scriptTotal/scriptNos（通过 `ScriptGroupV3Api.getScriptNoByGroupIds` 展开脚本组）。插入 `quartz_job`（job_type=TEMPLATE, job_status=EXEC），更新 content 中的 jobId。

### 调用链

```
TestTemplateController.saveTemplate
└─ TemplateService.saveTemplate(jsonStr) @Transactional
   ├─ ScriptGroupV3Api.getScriptNoByGroupIds → Script POST /v3/script/script_groups/script_no
   └─ QuartzJobMapper.insert → MySQL quartz_job
```

---

## 4. GET /v3/realweb/template/copy — 复制模板

### 入口

`TestTemplateController.copyTemplate(@NotNull @RequestParam("job_id") Integer jobId, @NotNull @RequestParam("user_id") Integer userId, @NotEmpty @RequestParam("user_name") String userName, @NotNull @RequestParam("dir_id") Integer dirId)`

### 请求参数

| 字段 | 来源 | 必填 | 说明 |
|------|------|------|------|
| job_id | Query | 是 | 源模板ID |
| user_id | Query | 是 | 操作用户ID |
| user_name | Query | 是 | 操作用户名 |
| dir_id | Query | 是 | 目标目录ID |

### 响应结构

`ResponseResult<BaseResponseDTO>`，data.result = 复制结果。

### 实现意图

读取源 `QuartzJob`，创建副本（jobDesc+"的副本"，新UUID jobName），插入 `quartz_job`。若 dirId>0 调用 [DirQuartzApi.addDirQuart](../其他ApiServlet/service-DirQuartzApi.md) 创建目录关联。若非 TEMPLATE 类型且 cron 有效，调用 `QuartzJobServiceImpl.addScheduler` 注册定时调度。

### 调用链

```
TestTemplateController.copyTemplate
└─ TemplateService.copyTemplate(jobId, userId, userName, dirId)
   ├─ QuartzJobMapper.selectById → MySQL quartz_job
   ├─ QuartzJobMapper.insert → MySQL quartz_job (新行)
   ├─ DirQuartzApi.addDirQuart → RealLogfile POST /v3/core/dir_quartz_job/add_dir_quartz_job
   └─ QuartzJobServiceImpl.addScheduler(jobId) → Quartz注册
```

---

## 5. POST /v3/realweb/template/update_template_content — 更新模板内容

### 入口

`TestTemplateController.updateSourceTag(@RequestBody TemplateContentRequestDTO requestDTO)`

### 请求参数（TemplateContentRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| tagList | List&lt;Integer&gt; | 否 | 标签ID列表 |
| jobIdList | List&lt;Integer&gt; | 是 | 模板ID列表（空抛 GeneralException） |
| editType | Integer | 是 | 编辑类型（TagEditEnum ADD/...；空抛 GeneralException） |

### 响应结构

`ResponseResult<BaseResponseDTO>`，data.result = 更新行数。

### 实现意图

`@Transactional`，遍历 jobIdList 中的模板，将 tagList 合并到 job_content JSON 的 `paramSource` → `tagList` 字段。

### 调用链

```
TestTemplateController.updateSourceTag
└─ TemplateService.updateTemplateContent(tagList, jobIdList, editType) @Transactional
   └─ QuartzJobMapper.update → MySQL quartz_job (job_content更新)
```

---

## 6. GET /v3/realweb/template/list_template_id — 查询模板ID列表

### 入口

`TestTemplateController.getQuartzJobId(@RequestParam("project_id") Integer projectId, @RequestParam("business_type") Integer businessType)`

### 请求参数

| 字段 | 来源 | 必填 | 说明 |
|------|------|------|------|
| project_id | Query | 是 | 项目ID |
| business_type | Query | 是 | 业务类型 |

### 响应结构

`ResponseResult<TemplateIdResponseDTO>`：
- `totalCount`: Integer
- `templateIds`: List&lt;Integer&gt;

### 实现意图

查询指定项目+业务类型下的所有有效模板ID和总数。

### 调用链

```
TestTemplateController.getQuartzJobId
└─ QuartzJobServiceImpl.getJobId(projectId, businessType) + countTemplate(...)
   └─ QuartzJobMapper → MySQL quartz_job
```

---

## 备注

- `listByCondition` 是唯一使用 `@Valid` 的 Controller 方法（projectId 非空校验）。
- `saveTemplate` 和 `updateSourceTag` 使用 `@Transactional`。
- `copyTemplate` 手动 @NotNull/@NotEmpty 校验。
- 模板与计划关联关系通过 `TestPlanV3Api` 远程查询 RealLogfile。

相关文档：[00-分支索引](00-分支索引.md)
