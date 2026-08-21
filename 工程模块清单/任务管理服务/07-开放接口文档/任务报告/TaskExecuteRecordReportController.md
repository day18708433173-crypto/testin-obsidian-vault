# TaskExecuteRecordReportController — 任务报告步骤与详情

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/task/TaskExecuteRecordReportController.java`
> 类级路由：`/real_task`
> Service 实现：`cn.testin.service.impl.task.TaskExecuteRecordReportServiceImpl`（1582 行）
> 业务：任务报告维度的步骤查询、步骤详情、执行报告详情。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 1 | POST | `/v3/real_task/report_steps` | getReportSteps | 查询任务报告的步骤列表 |
| 2 | POST | `/v3/real_task/report_step_detail` | getStepDetailByStepId | 查询步骤的详细执行信息 |
| 3 | GET | `/v3/real_task/{task_execute_record_id}/{task_execute_record_report_id}` | getTaskExecuteReportDetail | 查询任务执行报告详情 |

---

## 1. POST /v3/real_task/report_steps — 报告步骤列表

### 入口

`TaskExecuteRecordReportController.getReportSteps(@RequestBody TaskReportStepRequestDTO request)`

### 请求参数（TaskReportStepRequestDTO）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskExecuteRecordId | Integer | 是 | 执行记录id（服务层按此查询，不存在则报"任务不存在"） |
| taskExecuteRecordReportId | Long | 是 | 执行记录报告id（服务层按此查询，不存在则报"任务不存在"） |
| callTag | String | 否 | 执行轨迹标识 |
| retryNum | Integer | 否 | 重试次数 |
| stepId | Integer | 否 | 步骤id |

### 响应结构

`ResponseResult<TaskReportStepResponseDTO>`，含步骤列表及每步的执行统计。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.initStageList | JSONArray | 初始化阶段列表（TaskReportInitDTO） |
| data.initStageList.startTime | Long | 开始时间 |
| data.initStageList.endTime | Long | 结束时间 |
| data.initStageList.name | String | 名称 |
| data.initStageList.processName | String | 过程名称 |
| data.initStageList.totalTime | Long | 总时长 |
| data.initStageList.stage | String | 阶段 |
| data.initStageList.resultCode | Integer | 结果码 |
| data.initStageList.status | Integer | 状态 |
| data.recordReports | JSONArray | 执行记录报告列表（TaskExecuteRecordReportDTO） |
| data.recordReports.id | Long | 报告id |
| data.recordReports.taskExecuteRecordId | Integer | 执行记录id |
| data.recordReports.dataIdentifierId | String | 数据标识id |
| data.recordReports.taskExecuteRecordScriptId | Long | 脚本信息id |
| data.recordReports.taskExecuteRecordDeviceId | Long | 设备信息id |
| data.recordReports.scriptNo | Integer | 脚本No |
| data.recordReports.deviceId | String | 设备id |
| data.recordReports.orderNum | Integer | 执行顺序 |
| data.recordReports.rowId | Integer | 执行数据行id |
| data.recordReports.scriptInfo | JSONObject | 主脚本信息（TaskExecuteRecordScriptDTO） |
| data.recordReports.scriptInfo.id | Long | 脚本信息id |
| data.recordReports.scriptInfo.scriptType | Integer | 脚本类型：1脚本，2脚本组 |
| data.recordReports.scriptInfo.scriptExecuteType | Integer | 1 app脚本，3 web脚本，5 pc脚本 |
| data.recordReports.scriptInfo.scriptNo | Integer | 脚本No |
| data.recordReports.scriptInfo.orderNum | Integer | 执行数据顺序 |
| data.recordReports.scriptInfo.scriptId | Integer | 脚本id快照 |
| data.recordReports.scriptInfo.scriptName | String | 脚本名称 |
| data.recordReports.scriptInfo.scriptTags | String | 脚本关联的tag |
| data.recordReports.scriptInfo.scriptUrl | String | 脚本Url |
| data.recordReports.scriptInfo.scriptMd5 | String | 脚本Md5 |
| data.recordReports.scriptInfo.stepCount | Integer | 脚本步骤数量 |
| data.recordReports.callTag | String | 最外层的callTag |
| data.recordReports.taskStatus | Integer | 结果 |
| data.recordReports.retryNum | Integer | 重试次数 |
| data.recordReports.failCallTag | String | 失败的调用 tag |
| data.recordReports.failStepId | Integer | 失败的步骤 |
| data.recordReports.scriptNoWithStepDetail | JSONObject | 脚本No对应的步骤详情（Map<Integer, ScriptStepDetailDTO>，key为脚本No） |
| data.recordReports.scriptNoWithStepDetail.*.scriptId | Integer | 脚本id（JSON字段名 si） |
| data.recordReports.scriptNoWithStepDetail.*.stepCount | Integer | 步骤数（JSON字段名 sc） |
| data.steps | JSONArray | 步骤列表（TaskExecuteStepInfoDTO） |
| data.steps.stepId | Integer | 步骤id |
| data.steps.parentCallTag | String | 父调用 tag |
| data.steps.callTag | String | 调用 tag |
| data.steps.testResult | Integer | 测试结果 |
| data.steps.stepName | String | 步骤名 |
| data.steps.stepDescription | String | 步骤描述 |
| data.steps.stepCustomizeDescription | String | 步骤自定义描述 |
| data.steps.disable | Integer | 是否禁用 |
| data.steps.stepType | String | 步骤类型 |
| data.steps.totalTime | Long | 总时长 |
| data.steps.stepCount | Integer | 步骤数 |

### 实现意图

按报告维度分页查询执行步骤列表，每个步骤携带执行状态、耗时、设备等概览信息。

### 关联表

`task_execute_record_report_detail`

---

## 2. POST /v3/real_task/report_step_detail — 步骤详情

### 入口

`TaskExecuteRecordReportController.getStepDetailByStepId(@RequestBody TaskReportStepDetailRequestDTO request)`

### 请求参数（TaskReportStepDetailRequestDTO）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| taskExecuteRecordId | Integer | 是 | 执行记录id（服务层按此查询，不存在则报"任务不存在"） |
| taskExecuteRecordReportId | Long | 是 | 执行记录报告id（服务层按此查询，不存在则报"任务不存在"） |
| callTag | String | 否 | 执行轨迹标识 |
| retryNum | Integer | 否 | 重试次数 |
| stepId | Integer | 否 | 步骤id |

### 响应结构

`ResponseResult<TaskReportStepDetailResponseDTO>`，含步骤的完整执行日志、输入参数、输出值、截图、性能数据。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.stepInfo | JSONObject | 步骤信息（TaskReportStepInfoDTO） |
| data.stepInfo.stepId | Integer | 步骤id |
| data.stepInfo.startTime | Long | 步骤开始时间 |
| data.stepInfo.endTime | Long | 步骤结束时间 |
| data.stepInfo.description | String | 步骤描述 |
| data.stepInfo.action | String | 动作 |
| data.stepInfo.imageUrl | String | 结果截图路径 |
| data.stepInfo.imageKey | String | 结果截图 key |
| data.stepInfo.expectedImageKey | String | 期望截图 key |
| data.stepInfo.similarity | Integer | 截图相似度 |
| data.stepInfo.status | Integer | 状态 |
| data.stepInfo.resultCategory | Integer | 平台结果信息 |
| data.stepInfo.errorCode | Integer | 错误码 |
| data.stepInfo.errorMsg | String | 错误信息 |
| data.stepInfo.log | String | 错误日志 |
| data.stepInfo.logUrl | String | 日志链接 |
| data.stepInfo.anrTracesUrl | String | anr_traces 文件 |
| data.stepInfo.videoUrl | String | 视频链接 |
| data.stepInfo.afterImage | String | 出错后截图 |
| data.stepInfo.lineNum | Integer | 日志行号 |
| data.stepInfo.callTag | String | 执行轨迹标识 |
| data.stepInfo.network | Boolean | 是否有网络（True/False） |
| data.stepInfo.networkStrength | Integer | 网络强度（0~9） |
| data.stepInfo.skip | Integer | 是否忽略此步骤 |
| data.stepInfo.extension | String | 扩展信息（JSON 字符串） |
| data.stepInfo.caseId | String | 引用脚本id（对应 testcase 属性id） |
| data.stepInfo.monkeyInfo | JSONObject | monkey 信息（TaskReportStepMonkeyInfoDTO） |
| data.stepInfo.monkeyInfo.command | String | monkey 命令 |
| data.stepInfo.monkeyInfo.outputUrl | String | 输出文件地址 |
| data.stepInfo.monkeyInfo.anrTraceUrl | String | anr_trace 文件地址 |
| data.stepInfo.monkeyInfo.imageInfos | JSONArray | monkey 截图列表（TaskReportStepImageInfoDTO） |
| data.stepInfo.monkeyInfo.imageInfos[].timestamp | Long | 时间戳 |
| data.stepInfo.monkeyInfo.imageInfos[].action | String | 动作 |
| data.stepInfo.monkeyInfo.imageInfos[].tag | String | 标签 |
| data.stepInfo.monkeyInfo.imageInfos[].name | String | 名称 |
| data.stepInfo.monkeyInfo.imageInfos[].imageUrl | String | 截图地址 |
| data.stepInfo.monkeyInfo.imageInfos[].index | Integer | 序号 |
| data.stepInfo.monkeyInfo.imageInfos[].network | Boolean | 是否有网络 |
| data.stepInfo.monkeyInfo.imageInfos[].networkStrength | Integer | 网络强度 |
| data.stepInfo.capabilityInfo | JSONObject | 性能数据信息（TaskReportStepCapabilityInfoDTO） |
| data.stepInfo.capabilityInfo.perfSign | String | 统计点标记值 |
| data.stepInfo.capabilityInfo.name | String | 统计项名称 |
| data.stepInfo.capabilityInfo.actionTime | Long | 步骤执行当前时间 |
| data.stepInfo.capabilityInfo.networkUpFlow | Long | 当前上行流量 |
| data.stepInfo.capabilityInfo.networkDownFlow | Long | 当前下行流量 |
| data.stepInfo.capabilityInfo.actionTimeTotal | Long | 总耗时 |
| data.stepInfo.capabilityInfo.networkUpFlowTotal | Long | 总上行流量 |
| data.stepInfo.capabilityInfo.networkDownFlowTotal | Long | 总下行流量 |
| data.stepInfo.capabilityInfo.upNetflowExceed | Boolean | 上行流量是否超出达标值 |
| data.stepInfo.capabilityInfo.downNetflowExceed | Boolean | 下行流量是否超出达标值 |
| data.stepInfo.capabilityInfo.actionTimeExceed | Boolean | 耗时是否超出达标值 |
| data.stepInfo.capabilityInfo.actionTimeStandard | Long | 执行耗时达标值 |
| data.stepInfo.capabilityInfo.networkUpFlowStandard | Long | 上行流量达标值 |
| data.stepInfo.capabilityInfo.networkDownFlowStandard | Long | 下行流量达标值 |
| data.stepInfo.capabilityInfo.osCpu | Double | 系统CPU占用 |
| data.stepInfo.capabilityInfo.appCpu | Double | 应用CPU占用 |
| data.stepInfo.capabilityInfo.osRam | Long | 系统内存占用 |
| data.stepInfo.capabilityInfo.appRam | Long | 应用内存占用 |
| data.stepInfo.checkInfo | JSONObject | 检查点信息（TaskReportStepCheckInfoDTO） |
| data.stepInfo.checkInfo.name | String | 检查点名称 |
| data.stepInfo.checkInfo.sign | String | 指纹信息 |
| data.stepInfo.checkInfo.actionTime | Long | 检查点报错时间 |
| data.stepInfo.checkInfo.scriptNo | Integer | 脚本编号 |
| data.stepInfo.checkInfo.scriptId | Integer | 脚本id |
| data.stepInfo.checkInfo.deviceId | String | 设备id |
| data.stepInfo.checkInfo.errorMsg | String | 错误信息 |
| data.stepInfo.checkInfo.rule | String | 终止策略 |
| data.stepInfo.checkInfo.resultCategory | Integer | 分类 |
| data.stepInfo.checkInfo.scriptDescription | String | 脚本描述 |
| data.stepInfo.checkLogInfos | JSONArray | 日志检查点信息列表（TaskReportStepCheckLogInfoDTO） |
| data.stepInfo.checkLogInfos[].sign | String | 指纹信息 |
| data.stepInfo.checkLogInfos[].keyword | String | 关键字 |
| data.stepInfo.checkLogInfos[].rule | Integer | 规则（0不存在，1存在） |
| data.stepInfo.checkLogInfos[].line | Integer | 行号（-1不存在） |
| data.stepInfo.checkLogInfos[].passCheck | Integer | 通过状态（1通过，0不通过） |
| data.stepInfo.extensionFile | String | 步骤扩展文件 |
| data.stepInfo.runBeforeParam | JSONObject | 执行前参数 |
| data.stepInfo.runAfterParam | JSONObject | 执行后参数 |
| data.stepInfo.runBeforeParams | String | 执行前参数（带变量类型） |
| data.stepInfo.runAfterParams | String | 执行后参数（带变量类型） |
| data.stepInfo.aiFormExeResult | String | AI 表单执行结果 |
| data.stepInfo.warningTags | JSONArray | 告警标签列表（TaskReportWarningTagDTO） |
| data.stepInfo.warningTags[].warningCode | Integer | 错误码 |
| data.stepInfo.warningTags[].warningMsg | String | 错误信息 |
| data.stepInfo.warningTags[].warningTime | Long | 发生时间 |
| data.stepInfo.warningTags[].stepIndex | Integer | 错误发生步骤 |
| data.stepInfo.warningTags[].stepPath | String | 步骤所在脚本标记 |
| data.stepInfo.warningTags[].detail | String | 错误堆栈 |
| data.stepInfo.pid | Integer | 进程id |
| data.stepInfo.stackInfo | JSONObject | 堆栈信息 |
| data.logUrl | String | 日志链接 |
| data.scriptType | Integer | 脚本类型 |
| data.scriptNo | Integer | 脚本编号 |
| data.scriptId | Integer | 脚本id |

---

## 3. GET /v3/real_task/{task_execute_record_id}/{task_execute_record_report_id} — 执行报告详情

### 入口

`TaskExecuteRecordReportController.getTaskExecuteReportDetail(@PathVariable taskExecuteRecordId, @PathVariable taskExecuteRecordReportId)`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| task_execute_record_id | Integer | 是 | 执行记录id（路径变量） |
| task_execute_record_report_id | Long | 是 | 执行记录报告id（路径变量） |

### 响应结构

`ResponseResult<TaskExecuteReportDetailDTO>`，含执行记录的完整报告数据。

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 状态码，0 为成功 |
| msg | String | 提示信息 |
| data | Object | 业务数据 |
| data.taskBaseInfo | JSONObject | 任务基础信息（TaskExecuteRecordInfoDTO） |
| data.taskBaseInfo.id | Integer | 主键 |
| data.taskBaseInfo.parentId | Integer | 父节点id |
| data.taskBaseInfo.projectId | Integer | 项目id |
| data.taskBaseInfo.taskType | Integer | 任务类型：1 app，3 web，5 pc |
| data.taskBaseInfo.suiteId | Integer | 应用id |
| data.taskBaseInfo.taskName | String | 任务名称 |
| data.taskBaseInfo.taskStatus | Integer | 任务状态 |
| data.taskBaseInfo.executeRecordTaskId | Long | 关联测试计划的任务id |
| data.taskBaseInfo.executeRecordTaskName | String | 关联测试计划的任务名称 |
| data.taskBaseInfo.userId | Integer | 任务创建人 |
| data.taskBaseInfo.taskExecuteId | String | 任务执行id（uuid） |
| data.taskBaseInfo.deviceFocusScheduling | Integer | 设备配置集中调度 |
| data.taskBaseInfo.envId | Integer | 任务关联的环境id |
| data.taskBaseInfo.envConfig | String | 环境快照信息 |
| data.taskBaseInfo.level | Integer | 任务执行等级（越小优先级越高） |
| data.taskBaseInfo.appPackageId | Integer | 提测选择的app包id |
| data.taskBaseInfo.latestPackage | Integer | 是否使用最新包：0不使用，1使用 |
| data.taskBaseInfo.packageUrl | String | 直接使用的包url |
| data.taskBaseInfo.networkType | Integer | 网络类型：1 wifi，2 sim，3 模拟网络 |
| data.taskBaseInfo.systemPlatformId | Integer | 包关联的系统平台id |
| data.taskBaseInfo.appName | String | app名称 |
| data.taskBaseInfo.appPackageName | String | 包名称 |
| data.taskBaseInfo.appSize | Long | app提测大小 |
| data.taskBaseInfo.appBuild | String | app版本号 |
| data.taskBaseInfo.appIconUrl | String | app图标地址 |
| data.taskBaseInfo.appStartPath | String | app启动路径 |
| data.taskBaseInfo.appMd5 | String | app的md5信息 |
| data.taskBaseInfo.simulateNetworkName | String | 模拟网络名称（networkType=2时有效） |
| data.taskBaseInfo.networkConfig | String | 模拟网络快照信息 |
| data.taskBaseInfo.executeMethod | Integer | 执行方式：1分布式，2顺序执行 |
| data.taskBaseInfo.oldTaskId | Integer | 重测来源的老任务id |
| data.taskBaseInfo.retestStatus | String | 重测时需要重测的状态 |
| data.taskBaseInfo.dataSourceId | Integer | 关联的数据源id |
| data.taskBaseInfo.dataDistributeType | Integer | 数据下发类型：0按设备，1按脚本，2数据驱动 |
| data.taskBaseInfo.dataTag | String | 需要选择的数据标签 |
| data.taskBaseInfo.skipDataTag | String | 需要跳过的数据 |
| data.taskBaseInfo.finishCallBackUrl | String | 任务完成回调地址 |
| data.taskBaseInfo.callBackAdditional | String | 回调额外信息 |
| data.taskBaseInfo.coverInstall | Integer | 执行前卸载安装：1开启，0不开启 |
| data.taskBaseInfo.overwriteInstall | Integer | 执行前覆盖安装：1覆盖，0不覆盖 |
| data.taskBaseInfo.cleanData | Integer | 执行后清理数据：1清理，0不清理，-1用上位机配置 |
| data.taskBaseInfo.uninstall | Integer | 执行后不卸载app：1不卸载，0卸载 |
| data.taskBaseInfo.keepApp | Integer | 执行后关闭应用：1不关闭，0关闭 |
| data.taskBaseInfo.video | Integer | 是否录制视频：0不录制，1录制，2仅失败录制 |
| data.taskBaseInfo.resign | Integer | iOS重签配置：1重签，0不重签 |
| data.taskBaseInfo.taskExecuteMode | Integer | 上位机执行形式：1简单形式，2前端配置模式 |
| data.taskBaseInfo.terminationOnError | Integer | 脚本出错是否终止后续：0继续，1终止 |
| data.taskBaseInfo.stepGlobalTimeout | Long | 步骤全局超时时间（毫秒） |
| data.taskBaseInfo.customFilePath | String | 采集app输出日志的位置 |
| data.taskBaseInfo.logCollection | Integer | 是否记录日志：0关1开 |
| data.taskBaseInfo.performanceDataCollection | Integer | 是否记录性能数据：0关1开 |
| data.taskBaseInfo.traversalTime | Long | 遍历时长 |
| data.taskBaseInfo.monkeyTime | Long | monkey时长 |
| data.taskBaseInfo.retryNum | Integer | 脚本失败重试次数 |
| data.taskBaseInfo.appStepGlobalTimeOut | Integer | app脚本步骤全局超时时间 |
| data.taskBaseInfo.webStepGlobalTimeOut | Integer | web脚本步骤全局超时时间 |
| data.taskBaseInfo.pcStepGlobalTimeOut | Integer | pc脚本步骤全局超时时间 |
| data.script | JSONObject | 脚本信息（TaskExecuteRecordScript） |
| data.script.id | Long | 主键 |
| data.script.taskExecuteRecordId | Integer | 关联的任务id |
| data.script.scriptType | Integer | 1脚本，2脚本组 |
| data.script.scriptExecuteType | Integer | 1 app脚本，3 web脚本，5 pc脚本 |
| data.script.scriptNo | Integer | 脚本No（脚本组时为groupId） |
| data.script.orderNum | Integer | 执行数据顺序 |
| data.script.count | Integer | 当前脚本执行次数 |
| data.script.coverInstall | Integer | 执行前卸载安装：1卸载安装，0不卸载 |
| data.script.overwriteInstall | Integer | 执行前覆盖安装：1覆盖，0不覆盖 |
| data.script.cleanData | Integer | 执行后清理数据：1清理，0不清理 |
| data.script.keepApp | Integer | 执行后关闭应用：1不关闭，0关闭 |
| data.script.terminationOnError | Integer | 脚本出错是否终止后续：0继续，1终止 |
| data.script.createTime | Date | 创建时间 |
| data.script.updateTime | Date | 更新时间 |
| data.script.scriptId | Integer | 脚本id快照 |
| data.script.scriptName | String | 脚本名称 |
| data.script.scriptTags | String | 脚本关联的tag |
| data.script.scriptUrl | String | 脚本Url |
| data.script.scriptMd5 | String | 脚本Md5 |
| data.script.scriptInGroup | String | 脚本组下脚本id快照（脚本组去掉后无用） |
| data.script.caseId | Integer | 用例id |
| data.script.caseStepId | Integer | 用例步骤id |
| data.script.caseStepOrder | Integer | 用例步骤顺序 |
| data.script.preCaseStepIds | String | 前置步骤id列表 |
| data.script.aftCaseStepIds | String | 后置步骤id列表 |
| data.script.stepExpect | String | 步骤期望 |
| data.script.stepDesc | String | 步骤描述 |
| data.device | JSONObject | 设备信息（TaskExecuteRecordDevice） |
| data.device.id | Long | 主键 |
| data.device.taskExecuteRecordId | Integer | 关联的任务id |
| data.device.deviceId | String | 设备id（app为deviceId，web为ip_osName_type_version组合，pc为desktopId） |
| data.device.deviceSource | String | 设备云id |
| data.device.ucomId | String | 上位机id |
| data.device.deviceType | Integer | 设备类型：1 app，3 web，5 pc |
| data.device.createTime | Date | 创建时间 |
| data.device.updateTime | Date | 更新时间 |
| data.device.deviceStatus | Integer | 设备执行状态 |
| data.device.brandName | String | 品牌名称 |
| data.device.aliasName | String | 设备别名 |
| data.device.dpiWidth | Integer | 屏幕分辨率宽度 |
| data.device.dpiHeight | Integer | 屏幕分辨率高度 |
| data.device.serialNumber | String | 设备序列号 |
| data.device.systemPlatformName | String | 设备平台名称 |
| data.device.webDeviceTypeName | String | web浏览器类型名称 |
| data.device.webVersion | String | web浏览器版本 |
| data.device.osName | String | web浏览器操作系统 |
| data.device.taskExecuteRecordCaseStepId | Integer | 用例执行步骤id |
| data.device.ucomIp | String | 上位机ip |
| data.device.releaseVersion | String | 系统发布版本 |
| data.device.descr | String | 描述 |
| data.device.location | String | 位置 |
| data.device.modelName | String | 设备型号 |
| data.report | JSONObject | 报告信息（TaskExecuteRecordReport） |
| data.report.id | Long | 主键 |
| data.report.taskExecuteRecordId | Integer | 执行记录id |
| data.report.dataIdentifierId | String | 数据标识id |
| data.report.taskExecuteRecordScriptId | Long | 脚本信息id |
| data.report.taskExecuteRecordDeviceId | Long | 设备信息id |
| data.report.scriptNo | Integer | 脚本No |
| data.report.deviceId | String | 设备id |
| data.report.orderNum | Integer | 执行顺序 |
| data.report.rowId | Integer | 执行数据行id |
| data.report.dependencyRowIndex | Integer | 依赖行索引 |
| data.report.dataSourceConfigId | Integer | 数据源配置id |
| data.report.expireTime | Long | 过期时间 |
| data.report.executeStatus | Integer | 执行状态 |
| data.report.resultCategory | Integer | 结果分类 |
| data.report.errorMessage | String | 错误信息 |
| data.report.resultUrl | String | 结果地址 |
| data.report.executeStartTime | Date | 执行开始时间 |
| data.report.executeEndTime | Date | 执行结束时间 |
| data.report.executeCostTime | Long | 执行耗时 |
| data.report.scriptInitCostTime | Long | 脚本初始化耗时 |
| data.report.scriptStepCostTime | Long | 脚本步骤耗时 |
| data.report.scriptPreparationCostTime | Long | 脚本准备耗时 |
| data.report.resultLogUrl | String | 结果日志地址 |
| data.report.oldData | Integer | 旧数据标识 |
| data.report.retestNum | Integer | 重测次数 |
| data.report.errorCodeId | Integer | 错误码id |
| data.report.errorCodeName | String | 错误码名称 |
| data.report.matchTime | Date | 匹配时间 |
| data.report.matchCount | Integer | 匹配次数 |
| data.report.createTime | Date | 创建时间 |
| data.report.updateTime | Date | 更新时间 |
| data.report.deviceCpu | Integer | 设备CPU |
| data.report.deviceMemory | Integer | 设备内存 |
| data.report.deviceFlow | Integer | 设备流量 |
| data.report.deviceTemperature | Integer | 设备温度 |
| data.report.taskRecordReportCaseId | Long | 报告用例id |
| data.report.caseId | Integer | 用例id |
| data.report.caseStepId | Integer | 用例步骤id |
| data.report.caseStepOrder | Integer | 用例步骤顺序 |
| data.report.preCaseStepIds | String | 前置步骤id列表 |
| data.report.aftCaseStepIds | String | 后置步骤id列表 |
| data.report.scriptExecuteType | Integer | 脚本执行类型 |
| data.report.caseName | String | 用例名称 |
| data.report.scriptName | String | 脚本名称 |
| data.report.stepExpect | String | 步骤期望 |
| data.report.stepDesc | String | 步骤描述 |
| data.report.executeResultSummary | Integer | 执行结果汇总 |
| data.reportRunInfo | JSONObject | 报告运行信息（TaskReportRunInfoDTO） |
| data.reportRunInfo.ucomId | String | 上位机id |
| data.reportRunInfo.ucomVersion | String | 上位机版本 |
| data.reportRunInfo.jetlag | Long | 时差 |
| data.reportRunInfo.installPath | String | 安装目录信息 |
| data.reportRunInfo.installTime | Long | 安装时间 |
| data.reportRunInfo.startTime | Long | 启动时间 |
| data.reportRunInfo.uninstallTime | Long | 卸载时间 |
| data.reportRunInfo.execTime | Long | 执行耗时 |
| data.reportRunInfo.totalTime | Long | 运行总长 |
| data.reportRunInfo.testProcesses | JSONArray | 处理流程信息（TaskReportTestProcessDTO） |
| data.reportRunInfo.testProcesses[].name | String | 阶段名（launch/monkey/uninstall/install/execute/traversal） |
| data.reportRunInfo.testProcesses[].testProcessName | String | 该步骤对应的操作名 |
| data.reportRunInfo.testProcesses[].startTime | Long | 开始时间 |
| data.reportRunInfo.testProcesses[].endTime | Long | 结束时间 |
| data.reportRunInfo.testProcesses[].totalTime | Long | 总时间 |
| data.reportRunInfo.testProcesses[].stage | String | 阶段 |
| data.reportRunInfo.testProcesses[].resultCode | Integer | 结果码 |
| data.reportRunInfo.errorCode | Integer | 错误码 |
| data.reportRunInfo.errorMsg | String | 错误信息 |
| data.reportRunInfo.pfCode | Integer | 平台错误码 |
| data.reportRunInfo.runParam | String | 运行时使用的参数 |
| data.reportRunInfo.videoInfo | JSONObject | 视频信息（TaskReportVideoInfoDTO） |
| data.reportRunInfo.videoInfo.url | String | 视频地址 |
| data.reportRunInfo.videoInfo.errorCode | Integer | 视频上传失败错误码 |
| data.reportRunInfo.videoInfo.errorMsg | String | 视频上传失败错误信息 |
| data.reportRunInfo.stepInfo | JSONObject | 问题步骤信息（TaskReportStepInfoDTO，结构同 report_step_detail 的 data.stepInfo） |
| data.reportRunInfo.warningTags | JSONArray | 错误信息列表（TaskReportWarningTagDTO） |
| data.reportRunInfo.warningTags[].warningCode | Integer | 错误码 |
| data.reportRunInfo.warningTags[].warningMsg | String | 错误信息 |
| data.reportRunInfo.warningTags[].warningTime | Long | 发生时间 |
| data.reportRunInfo.warningTags[].stepIndex | Integer | 错误发生步骤 |
| data.reportRunInfo.warningTags[].stepPath | String | 步骤所在脚本标记 |
| data.reportRunInfo.warningTags[].detail | String | 错误堆栈 |
| data.reportRunInfo.fromWarningCode | Integer | 新的错误编码 |

## 涉及表汇总

| 表 | 说明 |
|----|------|
| `task_execute_record` | 执行记录主表 |
| `task_execute_record_report` | 执行报告 |
| `task_execute_record_report_detail` | 报告详细步骤 |
| `task_execute_record_report_case` | 报告用例结果 |
| `task_execute_record_case_step` | 用例步骤记录 |
