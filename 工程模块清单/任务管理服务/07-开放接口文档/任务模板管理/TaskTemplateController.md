# TaskTemplateController — 任务模板管理（CRUD / 批量操作 / 用例设备管理）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/task/TaskTemplateController.java`
> 类级路由：`/real_task`
> Service 实现：`cn.testin.service.impl.task.TaskTemplateServiceImpl`（1703 行）
> 业务：任务模板的完整生命周期管理——创建、编辑、查询、删除、暂停/恢复/批量删除/复制；模板下的用例和设备的查询与删除；用例ID状态同步。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 1 | POST | `/v3/real_task/task_template` | addTaskTemplate | 新增任务模板 |
| 2 | PUT | `/v3/real_task/task_template/{task_template_id}` | updateTaskTemplate | 全量更新任务模板 |
| 3 | POST | `/v3/real_task/task_template/update_task_template` | updateTaskTemplatePart | 批量部分更新任务模板 |
| 4 | GET | `/v3/real_task/task_template` | selectTaskTemplates | 分页查询任务模板列表 |
| 5 | DELETE | `/v3/real_task/task_template/{task_template_id}` | deleteTaskTemplate | 删除单个模板 |
| 6 | POST | `/v3/real_task/task_template/batch_pause` | batchPauseTaskTemplate | 批量暂停任务模板 |
| 7 | POST | `/v3/real_task/task_template/batch_resume` | batchResumeTaskTemplate | 批量恢复任务模板 |
| 8 | POST | `/v3/real_task/task_template/batch_delete` | batchDeleteTaskTemplate | 批量删除任务模板 |
| 9 | POST | `/v3/real_task/task_template/copy` | copyTaskTemplate | 复制任务模板 |
| 10 | GET | `/v3/real_task/task_template/{task_template_id}` | getTaskTemplateDetailById | 获取模板详情 |
| 11 | POST | `/v3/real_task/task_template/task_template_detail` | getTaskTemplateDetails | 批量获取模板详情 |
| 12 | POST | `/v3/real_task/task_template/task_template_detail_base` | getTaskTemplateDetailBase | 批量获取模板基础信息 |
| 13 | GET | `/v3/real_task/task_template/cases` | getTaskTemplateCasesByQuery | 查询模板下用例列表 |
| 14 | POST | `/v3/real_task/task_template/batch_delete_case` | batchDeleteTaskTemplateCase | 批量删除模板下的用例 |
| 15 | GET | `/v3/real_task/task_template/devices` | getTaskTemplateDevicesByQuery | 查询模板下设备列表 |
| 16 | POST | `/v3/real_task/task_template/batch_delete_device` | batchDeleteTaskTemplateDevice | 批量删除模板下的设备 |
| 17 | GET | `/v3/real_task/task_template/list_template_id` | getTaskTemplateId | 根据条件获取模板ID |
| 18 | POST | `/v3/real_task/sync_case_id_status` | syncCaseIdStatus | 同步用例ID状态 |

统一响应包装：`ResponseResult<T> { int code; String msg; T data }`。
GET 查询接口带 `@UnderlineToCamel`：下划线 query 参数自动转驼峰绑定 DTO。

---

## 1. POST /v3/real_task/task_template — 新增任务模板

### 入口

`TaskTemplateController.addTaskTemplate(@RequestBody @Valid TaskTemplateRequestDTO request)`

### 请求参数（TaskTemplateRequestDTO，JSON Body，@Valid）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 是 | 项目ID（@NotNull） |
| userId | Integer | 是 | 用户ID（@NotNull） |
| userName | String | 否 | 用户名 |
| eid | Integer | 否 | 企业ID，私有云始终为1 |
| taskName | String | 是 | 模板名称（service 校验非空） |
| taskDesc | String | 否 | 模板描述（目前只有用例模板用到） |
| taskType | Integer | 是 | 任务类型：1=App，3=Web，5=PC，100*=用例驱动（service 校验非空） |
| checkScript | Integer | 否 | 是否检查脚本 |
| scripts | JSONArray | 否 | 脚本列表（TaskScriptInfoDTO，非用例任务必传） |
| scripts[].scriptNo | Integer | 否 | 脚本编号 |
| scripts[].groupId | Integer | 否 | 脚本组ID |
| scripts[].count | Integer | 否 | 执行次数 |
| scripts[].scriptExecuteType | Integer | 否 | 脚本执行类型（1=App，3=Web，5=PC） |
| scripts[].scriptExecStandard | JSONObject | 否 | 脚本执行标准（ScriptExecStandardDTO） |
| scripts[].scriptExecStandard.terminationOnError | Integer | 否 | 脚本错误是否终止后续脚本 |
| scripts[].scriptExecStandard.overwriteInstall | Integer | 否 | 执行前覆盖安装 |
| scripts[].scriptExecStandard.coverInstall | Integer | 否 | 执行前卸载安装 |
| scripts[].scriptExecStandard.uninstallApp | Integer | 否 | 执行后卸载 app |
| scripts[].scriptExecStandard.clearData | Integer | 否 | 执行后清理数据 |
| scripts[].scriptExecStandard.keepApp | Integer | 否 | 执行后保持应用 |
| cases | JSONArray | 否 | 用例列表（CaseInfoDTO，用例驱动类型时使用） |
| cases[].caseId | Integer | 否 | 用例ID |
| cases[].caseName | String | 否 | 用例名称 |
| cases[].caseDirId | Integer | 否 | 用例目录ID |
| cases[].caseRemark | String | 否 | 用例备注 |
| cases[].caseStatus | Integer | 否 | 用例状态：1待评审/2待设计/3设计中/4已完成/5已废弃 |
| cases[].caseCheckStatus | Integer | 否 | 用例检查状态 |
| cases[].caseLevel | Integer | 否 | 用例级别 |
| cases[].caseTagList | JSONArray | 否 | 用例标签（String） |
| checkDevice | Integer | 否 | 是否检查设备 |
| taskDeviceCondition | JSONObject | 否 | 设备全选条件（TaskDeviceCondition） |
| taskDeviceCondition.ucomIp | String | 否 | 上位机IP |
| taskDeviceCondition.deviceId | String | 否 | 设备ID |
| taskDeviceCondition.deviceIds | JSONArray | 否 | 设备ID列表（String） |
| taskDeviceCondition.brandName | String | 否 | 品牌名称 |
| taskDeviceCondition.modelName | String | 否 | 型号/设备类型 |
| taskDeviceCondition.osName | Integer | 否 | 系统类型 |
| taskDeviceCondition.osNames | JSONArray | 否 | 系统类型列表（Integer） |
| taskDeviceCondition.modelAlias | String | 否 | 设备别名 |
| taskDeviceCondition.status | Integer | 否 | 设备状态 |
| taskDeviceCondition.statuses | JSONArray | 否 | 设备状态集合（Integer） |
| taskDeviceCondition.action | Integer | 否 | 设备动作（是否空闲） |
| taskDeviceCondition.type | Integer | 否 | 企业级1/项目2 |
| taskDeviceCondition.remark | String | 否 | 备注 |
| taskDeviceCondition.deviceType | Integer | 否 | 类型（1/3/5） |
| devices | JSONArray | 否 | 设备具体信息（TaskDeviceInfoDTO，taskDeviceCondition 为空时传） |
| devices[].cloud | String | 否 | 设备云 |
| devices[].deviceId | String | 否 | 设备ID（App=deviceId，Web=ip_osName_type_version，PC=desktopId） |
| devices[].deviceName | String | 否 | 设备名称 |
| devices[].source | String | 否 | 设备云id（taskType 为 3、5 时必传） |
| devices[].ucomId | String | 否 | 桌面上位机id（taskType 为 5 时必传） |
| devices[].deviceType | Integer | 否 | 设备类型（1=App，3=Web，5=PC） |
| updateDeviceType | Integer | 否 | 设备更新类型：1=追加，2=覆盖 |
| dataDistributeType | Integer | 否 | 数据分发类型：0=按设备分配，1=按脚本分配，2=数据驱动 |
| executeMethod | Integer | 否 | 执行方式：1=分布式执行，2=顺序执行 |
| execStandard | JSONObject | 否 | 任务及上位机执行标准（TaskExecStandardDTO） |
| execStandard.deviceFocusScheduling | Integer | 否 | 设备集中调度 |
| execStandard.stepFailPreCall | Integer | 否 | 步骤失败预调用 |
| execStandard.scriptFailRetestCount | Integer | 否 | 脚本失败重试次数 |
| execStandard.video | Integer | 否 | 是否录屏：0不录/1录/2仅失败录制 |
| execStandard.stepGlobalTimeOut | Double | 否 | 脚本步骤全局超时（秒） |
| execStandard.logCollection | Integer | 否 | 是否记录日志：0关/1全部/2未通过 |
| execStandard.performanceDataCollection | Integer | 否 | 是否记录性能数据：0关/1开 |
| execStandard.preCompleteCallBack | String | 否 | 预完成回调 |
| execStandard.customFilePath | String | 否 | 采集 app 日志的位置 |
| execStandard.taskInitConfig | Integer | 否 | 任务初始化配置：0/空=默认，1=前端配置 |
| execStandard.deviceOfflineConfig | Integer | 否 | 设备离线配置 |
| execStandard.skipTagIds | JSONArray | 否 | 跳过的数据行标签ID（Integer） |
| execStandard.appStepGlobalTimeOut | Double | 否 | 用例任务 app 步骤全局超时（秒） |
| execStandard.webStepGlobalTimeOut | Double | 否 | 用例任务 web 步骤全局超时（秒） |
| execStandard.pcStepGlobalTimeOut | Double | 否 | 用例任务 pc 步骤全局超时（秒） |
| dataSource | JSONObject | 否 | 任务数据源配置（TaskDataSourceInfoDTO） |
| dataSource.dataSourceId | Integer | 否 | 数据源ID |
| dataSource.dataDistributeType | Integer | 否 | 数据分发类型 |
| dataSource.tagList | JSONArray | 否 | 选取的标签列表（Integer） |
| dataSource.skipTagIds | JSONArray | 否 | 跳过的标签列表（Integer） |
| dataSource.editType | Integer | 否 | 编辑类型 |
| networks | Integer | 否 | 网络类型（taskType 为 App 时有用） |
| simulateNetworkName | String | 否 | 模拟网络名称 |
| suiteInfo | JSONObject | 否 | 应用信息（TaskSuiteInfoDTO，taskType 为 App 时有用） |
| suiteInfo.suiteId | Integer | 否 | 应用ID |
| suiteInfo.appPackageId | Integer | 否 | 提测绑定的包ID |
| suiteInfo.sysPlatformId | Integer | 否 | 系统平台ID（android=1/ios=2/鸿蒙=3/其他=99） |
| suiteInfo.latestPackage | Integer | 否 | 是否使用最新包 |
| suiteInfo.packageName | String | 否 | 包名 |
| suiteInfo.packageUrl | String | 否 | 直接使用的包 URL |
| suiteInfo.iconUrl | String | 否 | 图标信息 |
| suiteInfo.versionName | String | 否 | 版本号 |
| suiteInfo.versionRemark | String | 否 | 版本备注 |
| suiteInfo.appName | String | 否 | 名称 |
| quartzInfo | JSONObject | 否 | 定时任务配置（CronQuartzDTO） |
| quartzInfo.status | Integer | 否 | 状态（是否启用） |
| quartzInfo.cronQuartz | String | 否 | cron 表达式 |
| quartzInfo.executeCount | Integer | 否 | 已执行次数 |
| quartzInfo.executeTotalCount | Integer | 否 | 总执行次数 |
| envId | Integer | 否 | 环境ID（脚本含 SQL 步骤时） |
| taskNotice | JSONObject | 否 | 通知配置（TaskNoticeDTO） |
| taskNotice.taskEmailNotice.status | Integer | 否 | 邮件通知状态 |
| taskNotice.taskEmailNotice.userIds | JSONArray | 否 | 邮件通知用户ID（Integer） |
| taskNotice.scriptExecuteFail.status | Integer | 否 | 脚本失败通知状态 |
| taskNotice.scriptExecuteFail.sendStrategy | Integer | 否 | 发送策略 |
| taskNotice.scriptExecuteFail.scriptFailCount | Integer | 否 | 脚本失败次数阈值 |
| taskNotice.scriptExecuteFail.robotChannelIds | JSONArray | 否 | 机器人渠道ID |
| taskNotice.taskFinishNotice.status | Integer | 否 | 任务完成通知状态 |
| taskNotice.taskFinishNotice.minPassRate | Integer | 否 | 最低通过率 |
| taskNotice.taskFinishNotice.maxPassRate | Integer | 否 | 最高通过率 |
| taskNotice.taskFinishNotice.robotChannelIds | JSONArray | 否 | 机器人渠道ID（Integer） |
| level | Integer | 否 | 优先级，越小越优先 |
| callbackUrl | String | 否 | 任务完成后的回调地址 |
| additionalInfo | String | 否 | 任务完成后需要添加的数据 |
| updateByDeviceType | Integer | 否 | 用例模板按设备端更新设备标识 |
| appStepGlobalTimeOut | Double | 否 | 用例模板 app 步骤全局超时（秒） |
| webStepGlobalTimeOut | Double | 否 | 用例模板 web 步骤全局超时（秒） |
| pcStepGlobalTimeOut | Double | 否 | 用例模板 pc 步骤全局超时（秒） |
| containsAppScript | Boolean | 否 | 是否有 App 脚本 |
| containsWebScript | Boolean | 否 | 是否有 Web 脚本 |
| containsPcScript | Boolean | 否 | 是否有 PC 脚本 |
| dirId | Integer | 否 | 模板目录ID |
| taskTemplateIds | JSONArray | 否 | 模板ID列表（仅批量更新接口使用） |

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 新模板ID（Integer）。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 新模板ID |

### 实现意图

创建任务模板，包含脚本/设备/用例/定时/通知等配置。支持保存为模板（saveTemplate）或作为一次性配置。模板创建后默认为"执行中"状态。

### 调用链

```
TaskTemplateController.addTaskTemplate
└─ TaskTemplateServiceImpl.addTaskTemplate(@Transactional)
   ├─ TaskTemplate.transformTaskTemplateDTO (DTO → Entity)
   ├─ taskTemplateMapper.insert (主表 task_template)
   ├─ taskTemplateDetailMapper.insert (明细表 task_template_detail)
   ├─ taskTemplateScriptMapper.insert (脚本关联)
   ├─ taskTemplateDeviceMapper.insert (设备关联)
   ├─ taskTemplateCaseMapper.insert (用例关联)
   └─ taskTemplateNoticeMapper.insert (通知关联)
```

### 涉及表

- `task_template`
- `task_template_detail`
- `task_template_script`
- `task_template_device`
- `task_template_case`
- `task_template_notice`

---

## 2. PUT /v3/real_task/task_template/{task_template_id} — 更新任务模板

### 入口

`TaskTemplateController.updateTaskTemplate(@PathVariable taskTemplateId, @RequestBody @Valid TaskTemplateRequestDTO request)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_template_id | Integer | 是 | 模板ID（路径变量） |
| projectId | Integer | 是 | 项目ID（@NotNull） |
| userId | Integer | 是 | 用户ID（@NotNull） |
| taskType | Integer | 是 | 任务类型（service 校验非空） |
| taskName | String | 否 | 模板名称（更新时非空才覆盖） |
| taskDesc | String | 否 | 模板描述 |
| scripts | JSONArray | 否 | 脚本列表（非用例任务必传） |
| cases | JSONArray | 否 | 用例列表 |
| devices | JSONArray | 否 | 设备列表 |
| taskDeviceCondition | JSONObject | 否 | 设备全选条件 |
| suiteInfo | JSONObject | 否 | 应用信息 |
| dataSource | JSONObject | 否 | 数据源配置 |
| networks | Integer | 否 | 网络类型 |
| simulateNetworkName | String | 否 | 模拟网络名称 |
| taskNotice | JSONObject | 否 | 通知配置 |
| quartzInfo | JSONObject | 否 | 定时任务配置 |
| execStandard | JSONObject | 否 | 执行标准 |
| executeMethod | Integer | 否 | 执行方式 |
| dataDistributeType | Integer | 否 | 数据分发类型 |
| envId | Integer | 否 | 环境ID |
| level | Integer | 否 | 优先级 |
| callbackUrl | String | 否 | 回调地址 |
| additionalInfo | String | 否 | 附加信息 |
| dirId | Integer | 否 | 模板目录ID |

> 其余字段同「新增任务模板」TaskTemplateRequestDTO。

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 影响行数。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 影响行数 |

### 实现意图

全量更新模板（先删后插关联数据：脚本、设备、用例、通知）。`@Transactional` 保证原子性。

---

## 3. POST /v3/real_task/task_template/update_task_template — 批量部分更新

### 入口

`TaskTemplateController.updateTaskTemplatePart(@RequestBody @Valid TaskTemplateRequestDTO request)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskTemplateIds | JSONArray | 是 | 模板ID列表（Integer，为空时接口直接返回） |
| projectId | Integer | 是 | 项目ID（@NotNull） |
| userId | Integer | 是 | 用户ID（@NotNull） |
| taskType | Integer | 否 | 任务类型 |
| taskName | String | 否 | 模板名称（非空才更新） |
| taskDesc | String | 否 | 模板描述 |
| scripts | JSONArray | 否 | 脚本列表 |
| cases | JSONArray | 否 | 用例列表 |
| devices | JSONArray | 否 | 设备列表 |
| taskDeviceCondition | JSONObject | 否 | 设备全选条件 |
| suiteInfo | JSONObject | 否 | 应用信息 |
| dataSource | JSONObject | 否 | 数据源配置 |
| networks | Integer | 否 | 网络类型 |
| taskNotice | JSONObject | 否 | 通知配置 |
| quartzInfo | JSONObject | 否 | 定时任务配置 |
| execStandard | JSONObject | 否 | 执行标准 |
| executeMethod | Integer | 否 | 执行方式 |
| envId | Integer | 否 | 环境ID |
| level | Integer | 否 | 优先级 |
| callbackUrl | String | 否 | 回调地址 |
| additionalInfo | String | 否 | 附加信息 |
| dirId | Integer | 否 | 模板目录ID |

> 只更新传入的非空字段，其余字段同 TaskTemplateRequestDTO。

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 影响行数。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 影响行数 |

---

## 4. GET /v3/real_task/task_template — 查询模板列表

### 入口

`TaskTemplateController.selectTaskTemplates(TaskTemplateConditionRequestDTO request)` — `@UnderlineToCamel`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| project_id | Integer | 是 | 项目ID（checkProjectId） |
| eid | Integer | 否 | 企业ID |
| user_id | Integer | 否 | 用户ID |
| task_type | Integer | 否 | 任务类型 |
| ignore_task_type | Integer | 否 | 忽略任务类型 |
| cron_task | Integer | 否 | 定时任务筛选 |
| id | Integer | 否 | 模板ID |
| ids | JSONArray | 否 | 模板ID列表（Integer） |
| filter_ids | JSONArray | 否 | 过滤ID列表（Integer） |
| template_type | Integer | 否 | 模板类型：1=模板，2=定时任务 |
| task_name | String | 否 | 模糊搜索模板名称 |
| task_template_desc | String | 否 | 模板描述 |
| suite_id | Integer | 否 | 应用ID |
| task_template_status | Integer | 否 | 模板状态 |
| create_user_name | String | 否 | 创建人名称 |
| create_user_ids | JSONArray | 否 | 创建人ID列表（Integer） |
| start_create_time | Long | 否 | 创建开始时间 |
| end_create_time | Long | 否 | 创建结束时间 |
| task_has_suite_type | Integer | 否 | 模板包含的端类型 |
| need_script | Boolean | 否 | 是否需要脚本 |
| need_device | Boolean | 否 | 是否需要设备 |
| need_case | Boolean | 否 | 是否需要用例 |
| dir_id | Integer | 否 | 目录ID |
| order_by_col | String | 否 | 排序字段 |
| order_by_type | String | 否 | 排序方式 |
| page | Integer | 否 | 当前页（默认1） |
| page_size | Integer | 否 | 每页大小（默认10） |

### 响应结构

`ResponseResult<BasePageListResponseDTO<TaskTemplateResponseDTO>>`

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |
| data.list | JSONArray | 模板列表（TaskTemplateResponseDTO） |
| data.list[].id | Integer | 模板主键 |
| data.list[].projectId | Integer | 项目ID |
| data.list[].taskType | Integer | 任务类型 |
| data.list[].suiteId | Integer | 应用ID |
| data.list[].taskName | String | 模板名称 |
| data.list[].taskDesc | String | 任务描述 |
| data.list[].taskHasSuiteType | JSONArray | 模板包含的端类型（Integer） |
| data.list[].taskTemplateStatus | Integer | 任务状态 |
| data.list[].templateType | Integer | 1=模板，2=定时任务 |
| data.list[].cronExpression | String | cron 表达式 |
| data.list[].caseNum | Integer | 用例数量 |
| data.list[].errorCaseNum | Integer | 错误用例数量 |
| data.list[].deviceNum | Integer | 设备数量 |
| data.list[].abnormalDeviceNum | Integer | 异常设备数量 |
| data.list[].createUserId | Integer | 创建用户ID |
| data.list[].updateUserId | Integer | 更新用户ID |
| data.list[].deviceList | JSONArray | 测试计划设备信息（PlanTaskDeviceDTO） |
| data.list[].deviceList[].deviceId | String | 设备ID |
| data.list[].deviceList[].ip | String | 设备IP |
| data.list[].deviceList[].deviceStatus | Integer | 设备状态 |
| data.list[].deviceList[].deviceType | Integer | 设备类型 |
| data.list[].scriptList | JSONArray | 测试计划脚本信息（PlanTaskScriptDTO） |
| data.list[].scriptList[].scriptNo | Integer | 脚本编号 |
| data.list[].scriptList[].scriptGroupId | Integer | 脚本组ID |
| data.list[].scriptList[].executeCount | Integer | 执行次数 |
| data.list[].caseList | JSONArray | 测试计划用例信息（PlanTaskCaseDTO） |
| data.list[].caseList[].caseId | Integer | 用例ID |
| data.list[].caseList[].caseStatus | Integer | 用例状态 |
| data.list[].caseList[].executeCount | Integer | 执行次数 |
| data.list[].envId | Integer | 环境ID |
| data.list[].osType | Integer | 系统类型 |
| data.list[].appPackageId | Integer | app 包ID |
| data.list[].dataSourceId | Long | 数据源ID |
| data.list[].dataSourceName | String | 数据源名称 |
| data.list[].createTime | Date | 创建时间 |
| data.list[].updateTime | Date | 更新时间 |
| data.list[].relationTaskPlanCount | Integer | 关联测试计划数量 |
| data.list[].planInfos | JSONArray | 关联测试计划信息（PlanInfoDTO） |
| data.list[].planInfos[].id | Long | 计划ID |
| data.list[].planInfos[].planInfoName | String | 计划名称 |

---

## 5. DELETE /v3/real_task/task_template/{task_template_id} — 删除模板

### 入口

`TaskTemplateController.deleteTaskTemplate(@PathVariable taskTemplateId, @RequestParam deleteTaskRecord)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_template_id | Integer | 是 | 模板ID（路径变量） |
| delete_task_record | Integer | 否 | 0=不删关联执行记录，1=删除（默认0，Query） |

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 影响行数。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 影响行数 |

---

## 6. POST /v3/real_task/task_template/batch_pause — 批量暂停

### 入口

`batchPauseTaskTemplate(@RequestBody TaskTemplateConditionRequestDTO request)`

### 请求参数（TaskTemplateConditionRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 是 | 项目ID（checkProjectId） |
| userId | Integer | 是 | 用户ID（checkUserId） |
| eid | Integer | 否 | 企业ID |
| ids | JSONArray | 否 | 模板ID列表（Integer，批量操作目标） |
| taskType | Integer | 否 | 任务类型 |
| templateType | Integer | 否 | 模板类型 |
| taskName | String | 否 | 模板名称 |
| taskTemplateStatus | Integer | 否 | 模板状态 |
| suiteId | Integer | 否 | 应用ID |
| dirId | Integer | 否 | 目录ID |
| createUserIds | JSONArray | 否 | 创建人ID列表（Integer） |
| startCreateTime | Long | 否 | 创建开始时间 |
| endCreateTime | Long | 否 | 创建结束时间 |
| ignoreTaskType | Integer | 否 | 忽略任务类型 |
| cronTask | Integer | 否 | 定时任务筛选 |

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 影响行数。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 影响行数 |

### 实现意图

按条件筛选模板后批量修改状态为暂停。暂停状态下模板不会参与定时执行。

---

## 7. POST /v3/real_task/task_template/batch_resume — 批量恢复

### 入口

`batchResumeTaskTemplate(@RequestBody TaskTemplateConditionRequestDTO request)`

### 请求参数（TaskTemplateConditionRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 是 | 项目ID（checkProjectId） |
| userId | Integer | 是 | 用户ID（checkUserId） |
| eid | Integer | 否 | 企业ID |
| ids | JSONArray | 否 | 模板ID列表（Integer） |
| taskType | Integer | 否 | 任务类型 |
| templateType | Integer | 否 | 模板类型 |
| taskName | String | 否 | 模板名称 |
| taskTemplateStatus | Integer | 否 | 模板状态 |
| suiteId | Integer | 否 | 应用ID |
| dirId | Integer | 否 | 目录ID |
| createUserIds | JSONArray | 否 | 创建人ID列表（Integer） |
| startCreateTime | Long | 否 | 创建开始时间 |
| endCreateTime | Long | 否 | 创建结束时间 |
| ignoreTaskType | Integer | 否 | 忽略任务类型 |
| cronTask | Integer | 否 | 定时任务筛选 |

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 影响行数。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 影响行数 |

---

## 8. POST /v3/real_task/task_template/batch_delete — 批量删除

### 入口

`TaskTemplateController.batchDeleteTaskTemplate(@RequestBody TaskTemplateDeleteRequestDTO request)`

### 请求参数（TaskTemplateDeleteRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 是 | 项目ID（checkProjectId） |
| userId | Integer | 是 | 用户ID（checkUserId） |
| ids | JSONArray | 否 | 模板ID列表（Integer，删除目标） |
| deleteRecord | Integer | 否 | 是否删除关联执行记录：0=否，1=是 |
| ignoreTaskType | Integer | 否 | 忽略任务类型 |
| eid | Integer | 否 | 企业ID |
| taskType | Integer | 否 | 任务类型 |
| templateType | Integer | 否 | 模板类型 |
| taskName | String | 否 | 模板名称 |
| taskTemplateStatus | Integer | 否 | 模板状态 |
| suiteId | Integer | 否 | 应用ID |
| dirId | Integer | 否 | 目录ID |

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 删除数量。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 删除数量 |

---

## 9. POST /v3/real_task/task_template/copy — 复制模板

### 入口

`TaskTemplateController.copyTaskTemplate(@RequestBody TaskTemplateCopyRequestDTO request)`

### 请求参数（TaskTemplateCopyRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 是 | 项目ID（@NotNull） |
| userId | Integer | 是 | 用户ID（@NotNull） |
| userName | String | 否 | 用户名 |
| eid | Integer | 否 | 企业ID |
| taskTemplateId | Integer | 否 | 被复制的模板ID |
| targetProjectId | Integer | 否 | 复制到的项目ID |

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 新模板ID。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 新模板ID |

### 实现意图

深拷贝任务模板（名称、脚本、设备、用例、通知配置全部复制到新模板）。

---

## 10. GET /v3/real_task/task_template/{task_template_id} — 模板详情

### 入口

`TaskTemplateController.getTaskTemplateDetailById(@PathVariable taskTemplateId)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_template_id | Integer | 是 | 模板ID（路径变量） |

### 响应结构

`ResponseResult<TaskTemplateDetailResponseDTO>`，含模板完整详情（脚本/设备/用例/通知全量关联数据）。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.id | Integer | 模板ID |
| data.projectId | Integer | 项目ID |
| data.taskName | String | 任务名称 |
| data.taskType | Integer | 任务类型 |
| data.dirId | Integer | 模板所属目录ID |
| data.scripts | JSONArray | 脚本列表（TaskScriptDetailDTO） |
| data.scripts[].scriptId | Integer | 脚本ID |
| data.scripts[].name | String | 脚本名称 |
| data.scripts[].tags | JSONArray | 脚本标签（String） |
| data.scripts[].scriptNo | Integer | 脚本编号 |
| data.scripts[].groupId | Integer | 脚本组ID |
| data.scripts[].count | Integer | 执行次数 |
| data.scripts[].scriptExecuteType | Integer | 脚本执行类型 |
| data.scripts[].checkStatus | Integer | 脚本检查状态 |
| data.scripts[].scriptUrl | String | 脚本URL |
| data.scripts[].scriptMd5 | String | 脚本MD5 |
| data.cases | JSONArray | 用例列表（CaseInfoDTO） |
| data.cases[].caseId | Integer | 用例ID |
| data.cases[].caseName | String | 用例名称 |
| data.cases[].caseStatus | Integer | 用例状态 |
| data.cases[].caseLevel | Integer | 用例级别 |
| data.cases[].caseTagList | JSONArray | 用例标签（String） |
| data.taskDeviceCondition | JSONObject | 设备全选条件（TaskDeviceCondition） |
| data.devices | JSONArray | 设备列表（TaskDeviceDetailDTO） |
| data.devices[].deviceId | String | 设备ID |
| data.devices[].deviceName | String | 设备名称 |
| data.devices[].status | Integer | 设备状态 |
| data.devices[].action | Integer | 设备动作 |
| data.devices[].brandName | String | 品牌名称 |
| data.devices[].modelName | String | 型号 |
| data.devices[].systemName | String | 系统名称 |
| data.devices[].deviceType | Integer | 设备类型 |
| data.dataDistributeType | Integer | 数据分发类型 |
| data.executeMethod | Integer | 执行方式 |
| data.execStandard | JSONObject | 执行标准（TaskExecStandardDTO） |
| data.dataSource | JSONObject | 数据源（TaskDataSourceInfoDTO） |
| data.networks | Integer | 网络类型 |
| data.simulateNetworkName | String | 模拟网络名称 |
| data.suiteInfo | JSONObject | 应用信息（TaskSuiteInfoDTO） |
| data.quartzInfo | JSONObject | 定时任务配置（CronQuartzDTO） |
| data.envId | Integer | 环境ID |
| data.taskNotice | JSONObject | 通知配置（TaskNoticeDTO） |
| data.callbackUrl | String | 回调地址 |
| data.additionalInfo | String | 附加信息 |
| data.taskHasSuiteType | JSONArray | 模板包含的端类型（Integer） |
| data.createUserId | Integer | 创建人ID |
| data.updateUserId | Integer | 更新人ID |
| data.createTime | Long | 创建时间 |
| data.updateTime | Long | 更新时间 |

> 说明：`data.taskDeviceCondition`、`data.execStandard`、`data.dataSource`、`data.suiteInfo`、`data.quartzInfo`、`data.taskNotice` 为嵌套对象，字段含义与「1. 创建任务模板」请求参数中的同名嵌套对象一致（TaskDeviceCondition / TaskExecStandardDTO / TaskDataSourceInfoDTO / TaskSuiteInfoDTO / CronQuartzDTO / TaskNoticeDTO）。

---

## 11. POST /v3/real_task/task_template/task_template_detail — 批量获取模板详情

### 入口

`TaskTemplateController.getTaskTemplateDetails(@RequestBody TaskTemplateDetailRequestDTO request)`

### 请求参数（TaskTemplateDetailRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| ids | JSONArray | 否 | 模板ID列表（Integer） |
| projectId | Integer | 否 | 项目ID |
| needBaseInfo | Boolean | 否 | 是否需要基础信息 |
| needScriptDetail | Boolean | 否 | 是否需要脚本详情 |
| needDeviceDetail | Boolean | 否 | 是否需要设备详情 |
| ignoreCaseStatus | Boolean | 否 | 是否忽略用例状态 |
| needCaseTagInfo | Boolean | 否 | 是否需要用例 tag 信息 |
| needCase | Boolean | 否 | 是否需要用例 |
| needScript | Boolean | 否 | 是否需要脚本 |
| needDevice | Boolean | 否 | 是否需要设备 |

### 响应结构

`ResponseResult<List<TaskTemplateDetailResponseDTO>>`，`data` = 模板详情数组。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | JSONArray | 模板详情列表（TaskTemplateDetailResponseDTO） |
| data[].id | Integer | 模板ID |
| data[].projectId | Integer | 项目ID |
| data[].taskName | String | 任务名称 |
| data[].taskType | Integer | 任务类型 |
| data[].scripts | JSONArray | 脚本列表 |
| data[].cases | JSONArray | 用例列表 |
| data[].devices | JSONArray | 设备列表 |
| data[].taskDeviceCondition | JSONObject | 设备全选条件 |
| data[].execStandard | JSONObject | 执行标准 |
| data[].dataSource | JSONObject | 数据源 |
| data[].suiteInfo | JSONObject | 应用信息 |
| data[].quartzInfo | JSONObject | 定时任务配置 |
| data[].taskNotice | JSONObject | 通知配置 |
| data[].createUserId | Integer | 创建人ID |
| data[].updateUserId | Integer | 更新人ID |
| data[].createTime | Long | 创建时间 |
| data[].updateTime | Long | 更新时间 |

> 各子字段含义同「模板详情」。

---

## 12. POST /v3/real_task/task_template/task_template_detail_base — 批量获取模板基础信息

### 入口

`TaskTemplateController.getTaskTemplateDetailBase(@RequestBody TaskTemplateDetailRequestDTO request)`

### 请求参数（TaskTemplateDetailRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| ids | JSONArray | 否 | 模板ID列表（Integer） |
| projectId | Integer | 否 | 项目ID |
| needBaseInfo | Boolean | 否 | 是否需要基础信息 |
| needScriptDetail | Boolean | 否 | 是否需要脚本详情 |
| needDeviceDetail | Boolean | 否 | 是否需要设备详情 |
| ignoreCaseStatus | Boolean | 否 | 是否忽略用例状态 |
| needCaseTagInfo | Boolean | 否 | 是否需要用例 tag 信息 |
| needCase | Boolean | 否 | 是否需要用例 |
| needScript | Boolean | 否 | 是否需要脚本 |
| needDevice | Boolean | 否 | 是否需要设备 |

### 响应结构

`ResponseResult<List<TaskTemplateDetailResponseDTO>>`（不含脚本/设备等关联明细）。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | JSONArray | 模板基础信息列表（TaskTemplateDetailResponseDTO） |
| data[].id | Integer | 模板ID |
| data[].projectId | Integer | 项目ID |
| data[].taskName | String | 任务名称 |
| data[].taskType | Integer | 任务类型 |
| data[].createUserId | Integer | 创建人ID |
| data[].updateUserId | Integer | 更新人ID |
| data[].createTime | Long | 创建时间 |
| data[].updateTime | Long | 更新时间 |

---

## 13. GET /v3/real_task/task_template/cases — 查询模板下用例列表

### 入口

`TaskTemplateController.getTaskTemplateCasesByQuery(TaskTemplateCaseQueryRequestDTO request)` — `@UnderlineToCamel`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| project_id | Integer | 是 | 项目ID（checkProjectId） |
| user_id | Integer | 是 | 用户ID（checkUserId） |
| eid | Integer | 否 | 企业ID |
| task_template_id | Integer | 否 | 模板ID |
| case_ids | JSONArray | 否 | 用例ID列表（Integer） |
| case_name | String | 否 | 用例名称 |
| case_id | Integer | 否 | 用例ID |
| case_create_user_name | String | 否 | 用例创建人 |
| case_update_user_name | String | 否 | 用例更新人 |
| case_level | Integer | 否 | 用例级别 |
| case_tag_list | String | 否 | 用例标签 |
| case_status | Integer | 否 | 用例状态 |
| page | Integer | 否 | 当前页 |
| page_size | Integer | 否 | 每页大小 |

### 响应结构

`ResponseResult<BasePageListResponseDTO<CaseInfoDTO>>`

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |
| data.list | JSONArray | 用例列表（CaseInfoDTO） |
| data.list[].caseId | Integer | 用例ID |
| data.list[].projectId | Integer | 项目ID |
| data.list[].dataSourceId | Long | 用例数据源ID |
| data.list[].dataSourceName | String | 用例数据源名称 |
| data.list[].caseName | String | 用例名称 |
| data.list[].caseDirId | Integer | 目录ID |
| data.list[].caseRemark | String | 备注 |
| data.list[].caseStatus | Integer | 用例状态：1待评审/2待设计/3设计中/4已完成/5已废弃 |
| data.list[].caseCheckStatus | Integer | 用例检查状态 |
| data.list[].caseLevel | Integer | 用例级别 |
| data.list[].caseTagList | JSONArray | 用例标签（String） |
| data.list[].caseSteps | JSONArray | 用例步骤（CaseStepInfoDTO） |
| data.list[].createUserId | Integer | 创建人ID |
| data.list[].updateUserId | Integer | 更新人ID |
| data.list[].scriptTypes | JSONArray | 脚本类型（Integer） |
| data.list[].status | Integer | 用例状态：0逻辑删除/1正常 |
| data.list[].caseVersion | String | 用例版本 |
| data.list[].errorStepNum | Integer | 错误步骤数 |

---

## 14. POST /v3/real_task/task_template/batch_delete_case — 批量删除模板用例

### 入口

`TaskTemplateController.batchDeleteTaskTemplateCase(@RequestBody TaskTemplateCaseDeleteRequestDTO request)`

### 请求参数（TaskTemplateCaseDeleteRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 是 | 项目ID（checkProjectId） |
| userId | Integer | 是 | 用户ID（checkUserId） |
| eid | Integer | 否 | 企业ID |
| taskTemplateId | Integer | 否 | 模板ID |
| caseIds | JSONArray | 否 | 用例ID列表（Integer） |
| caseName | String | 否 | 用例名称 |
| caseId | Integer | 否 | 用例ID |
| caseCreateUserName | String | 否 | 用例创建人 |
| caseUpdateUserName | String | 否 | 用例更新人 |
| caseLevel | Integer | 否 | 用例级别 |
| caseTagList | JSONArray | 否 | 用例标签列表（String） |
| caseStatus | Integer | 否 | 用例状态 |

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 删除数量。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 删除数量 |

---

## 15. GET /v3/real_task/task_template/devices — 查询模板下设备列表

### 入口

`TaskTemplateController.getTaskTemplateDevicesByQuery(TaskTemplateDeviceQueryRequestDTO request)` — `@UnderlineToCamel`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| project_id | Integer | 是 | 项目ID（checkProjectId） |
| user_id | Integer | 是 | 用户ID（checkUserId） |
| eid | Integer | 否 | 企业ID |
| task_template_id | Integer | 否 | 模板ID |
| device_ids | JSONArray | 否 | 设备ID列表（String） |
| device_id | String | 否 | 设备ID |
| device_type | Integer | 否 | 设备类型 |
| device_status | Integer | 否 | 设备状态 |
| ucom_ip | String | 否 | 上位机IP |
| device_desc | String | 否 | 设备描述 |
| device_os_type | Integer | 否 | 设备系统类型 |
| brand_name | String | 否 | 品牌名称 |
| page | Integer | 否 | 当前页 |
| page_size | Integer | 否 | 每页大小 |

### 响应结构

`ResponseResult<BasePageListResponseDTO<TaskDeviceDetailDTO>>`

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.page | Integer | 当前页 |
| data.pageSize | Integer | 每页大小 |
| data.totalPage | Integer | 总页数 |
| data.totalRow | Long | 总行数 |
| data.list | JSONArray | 设备列表（TaskDeviceDetailDTO） |
| data.list[].deviceId | String | 设备ID |
| data.list[].deviceName | String | 设备名称 |
| data.list[].status | Integer | 设备状态 |
| data.list[].action | Integer | 动作：0空闲/1测试/2真机调试 |
| data.list[].ip | String | 设备IP |
| data.list[].location | String | 设备所在 |
| data.list[].systemName | String | 系统名称 |
| data.list[].brandName | String | 品牌名称 |
| data.list[].modelName | String | 模型 |
| data.list[].modelAlias | String | 别名 |
| data.list[].releaseVersion | String | 版本 |
| data.list[].dpiHeight | Integer | 分辨率高 |
| data.list[].dpiWidth | Integer | 分辨率宽 |
| data.list[].descr | String | 设备备注 |
| data.list[].osName | Integer | 系统类型 |
| data.list[].networkType | Integer | 网络类型：0无网/1wifi/2mobile |
| data.list[].network | Integer | 0无网/1有网 |
| data.list[].webDeviceType | Integer | web 设备类型 |
| data.list[].webDeviceTypeName | String | web 设备资源类型名称 |
| data.list[].source | String | 设备云id |
| data.list[].ucomId | String | 上位机id |
| data.list[].deviceType | Integer | 设备类型 |

---

## 16. POST /v3/real_task/task_template/batch_delete_device — 批量删除模板设备

### 入口

`TaskTemplateController.batchDeleteTaskTemplateDevice(@RequestBody TaskTemplateDeviceQueryRequestDTO request)`

### 请求参数（TaskTemplateDeviceQueryRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| projectId | Integer | 是 | 项目ID（checkProjectId） |
| userId | Integer | 是 | 用户ID（checkUserId） |
| eid | Integer | 否 | 企业ID |
| taskTemplateId | Integer | 否 | 模板ID |
| deviceIds | JSONArray | 否 | 设备ID列表（String） |
| deviceId | String | 否 | 设备ID |
| deviceType | Integer | 否 | 设备类型 |
| deviceStatus | Integer | 否 | 设备状态 |
| ucomIp | String | 否 | 上位机IP |
| deviceDesc | String | 否 | 设备描述 |
| deviceOsType | Integer | 否 | 设备系统类型 |
| brandName | String | 否 | 品牌名称 |

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 删除数量。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 删除数量 |

---

## 17. GET /v3/real_task/task_template/list_template_id — 获取模板ID

### 入口

`TaskTemplateController.getTaskTemplateId(TaskTemplateConditionRequestDTO)` — `@UnderlineToCamel`

### 请求参数（Query）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| project_id | Integer | 是 | 项目ID（checkProjectId） |
| eid | Integer | 否 | 企业ID |
| task_type | Integer | 否 | 任务类型 |
| template_type | Integer | 否 | 模板类型 |
| task_name | String | 否 | 模板名称 |
| task_template_status | Integer | 否 | 模板状态 |
| suite_id | Integer | 否 | 应用ID |
| dir_id | Integer | 否 | 目录ID |
| ids | JSONArray | 否 | 模板ID列表（Integer） |
| create_user_ids | JSONArray | 否 | 创建人ID列表（Integer） |
| start_create_time | Long | 否 | 创建开始时间 |
| end_create_time | Long | 否 | 创建结束时间 |
| ignore_task_type | Integer | 否 | 忽略任务类型 |

### 响应结构

`ResponseResult<TaskTemplateIdDTO>`，返回符合条件的模板ID列表。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.totalCount | Integer | 模板总数 |
| data.templateIds | JSONArray | 模板ID列表（Integer） |

---

## 18. POST /v3/real_task/sync_case_id_status — 同步用例ID状态

### 入口

`TaskTemplateController.syncCaseIdStatus(@RequestBody TaskTemplateCaseStatusSyncRequestDTO request)`

### 请求参数（TaskTemplateCaseStatusSyncRequestDTO，JSON Body）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| caseId | Integer | 否 | 用例ID |
| caseCheckStatus | Integer | 否 | 用例检查状态 |

### 响应结构

`ResponseResult<BaseResultResponseDTO>`，`data.result` = 影响行数。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.result | Integer | 影响行数 |

### 实现意图

外部调用接口，同步用例在任务模板中的关联状态（用例关联/解关联模板后的状态更新）。

### 涉及表

- `task_template_case_status_sync`

## 涉及表汇总

| 表 | 说明 |
|----|------|
| `task_template` | 任务模板主表 |
| `task_template_detail` | 模板执行详细配置 |
| `task_template_script` | 模板-脚本关联 |
| `task_template_device` | 模板-设备关联 |
| `task_template_case` | 模板-用例关联 |
| `task_template_notice` | 模板-通知配置关联 |
| `task_template_standard_detail` | 模板-执行标准关联 |
| `task_template_case_status_sync` | 用例ID状态同步 |
