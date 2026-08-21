# MongoDB 集合 — PmReportDetail

> 数据库：pmweb_db 或 pmpc_db | 集合：`pmwebReportDetail_00~99` / `PmpcReportDetail_00~99`
> DAO：`PmReportDetailDAOImpl` (implements `IPmReportDetailDAO`)
> 实体：`cn.testin.realweb.pojo.mongo.PmReportDetail`

## 分片规则

`PmReportDetail.table(taskid)` → `BasePojo.getTable(taskid, "pmwebReportDetail" 或 "PmpcReportDetail")`：

- taskId 末 2 位数字 → `_00` ~ `_99`
- taskId 以 `pt` 开头 → pmpc_db（`PmpcReportDetail_XX`），否则 → pmweb_db（`pmwebReportDetail_XX`）

详见 [存储总览](../../web-pc处理服务/02-技术架构/00-存储总览.md)。

## 用途

报告详情：每个脚本执行结果（子子任务粒度）一条文档，是字段最多、读写最频繁的集合。承载测试结果、日志地址、截图/视频、账号与参数快照、数据驱动行号、补测/重试标记等。查询普遍以 `retryNum in (null, 0)`、`retestMark in (null, 0)` 过滤掉重试/补测历史记录。

## 主要字段

| 字段 | 类型 | 说明 |
|------|------|------|
| id | String | MongoDB 文档id |
| taskid | String | 任务ID（分片键、查询主条件） |
| subtaskid | String | 子任务ID（设备/浏览器粒度） |
| subsubtaskid | String | 脚本任务ID（脚本粒度，get/update 主条件） |
| vhost / eid / projectid / userid | Integer | 服务号/企业/项目组/用户 |
| testType | Integer | 测试类型 |
| scriptUrl / scriptTags / scriptDescr | String | 脚本地址/标签/描述 |
| scriptNo | Integer | 脚本no |
| groupId | Integer | 脚本组 |
| logurl | String | 日志下载地址 |
| resultCategory | Integer | 结果分类（ResultCategoryEnum，>PASS 即不通过） |
| accountId / accountPwd / accountExtension | String | 执行使用的账号/密码/扩展信息 |
| globalParam / localParam | String | 全局/局部参数 |
| reportdevice | ReportDevice | 设备信息（reportDetail 包） |
| testCases | List\<TestCase\> | 功能测试结果信息 |
| runinfo | ReportRunInfo | 运行信息 |
| logsummary | ReportLogSummary | 日志详情 |
| logSummaryUrl | String | 日志详情地址 |
| reporttime | Long | 报告结果时间 |
| execStandard | String | 执行策略（fast/normal/simple/monkey/script） |
| testresultUrl | String | 测试结果url |
| orderNum | Integer | 脚本顺序 |
| round | Integer | 任务执行轮数 |
| network | String | 网络类型 |
| tcpdumpUrl / processUrl / robotiumUrl / uiautomatorUrl / testprocessUrl / anrtracesUrl | String | 各类过程日志下载地址 |
| retryNum | Integer | 出错重试顺序（0/null=最后一次或未重试） |
| matchtime | Long | 任务匹配时间 |
| precompletetime | Long | 任务预完成时间 |
| retestMark | Integer | 补测标识（1=补测数据） |
| reTestNum | Integer | 脚本重测次数 |
| outputParams / inputParams / originalInputParams | String | 子子任务运行后/前/原始输入的全局变量 |
| ciccDyeUrl | String | 中金代码染色报告地址（脚本级） |
| dataRow | Integer | 使用数据表的第几行 |
| scriptDataRow | Map\<Integer,Integer\> | 各子脚本使用数据表的行号 |
| scriptDataUuid / rawDataUuid / scriptDataDetailUuid | String | 数据驱动前端行/次数/脚本对应uuid |
| rowId | Integer | 数据源对应的rowId |
| scripts | List\<ScriptInfo\> | 脚本列表（scriptSummary 包） |
| sourceConfigId / sourceConfigName / sourceConfigParentId | Integer/String | 数据表实例id/名称/父文件夹id |
| errorCauseTypeId | Integer | 错误原因类型id（自定义错误分析） |
| uuid | String | uuid |
| status / updatetime | Integer/Long | 继承 BasePojo |

静态方法 `generateReportDetail(reportDetail, taskinfoNode)`：从 TaskInfoNode 拷贝基本信息生成报告详情。子结构位于 `cn.testin.realweb.pojo.mongo.reportDetail` 包（ReportDevice、TestCase、ReportRunInfo、ReportLogSummary、TaskInfoNode 等）。

## 核心操作

| DAO 方法 | 操作 | 调用者 |
|----------|------|--------|
| insert(reportDetail) | 插入单条 | ReportServiceImpl（接收上报结果） |
| get(taskid, subsubtaskid) | 按 taskid+subsubtaskid 查单条（排除重试记录） | ReportServiceImpl, NoticeServiceImpl |
| list(taskid, conditionMap, keywords) | 条件查询列表，支持 subtaskid(s)/subsubtaskid/orderNum/retestMark/history 过滤与字段裁剪 | ReportServiceImpl, TaskSummaryServiceImpl, GenerateReportServiceImpl, NoticeServiceImpl, ScriptSummaryServiceImpl |
| baseList(conditionMap, page, pageSize) | 分页查询，支持 resultCategorys 与 errorCauseTypeIds 的 OR 组合、时间范围、onlyReTest、排序、字段排除 | ReportServiceImpl, NoticeServiceImpl |
| details(taskid, subtaskid, subsubtaskids, keywords, sort, history, containRetest) | 按子子任务数组批量查询（可含重试/补测） | ReportServiceImpl, GenerateReportServiceImpl |
| update(pmReportDetail) | 按 taskid+subsubtaskid 更新 resultCategory/originalInputParams/sourceConfig*/inputParams/errorCauseTypeId | ReportServiceImpl（结果分析、错误原因标注） |
| delete(taskid, subsubtaskid[, retryNum, matchtime]) | 删除报告（重试时删旧记录） | ReportServiceImpl |
| resetToRetestMark(taskid, subtaskid, subsubtaskid, retestMark, history) | 批量重置补测标记 | 补测流程 |
| getReportByResultCategory(taskId, [subSubTaskIds], resultCategories, scriptNos) | 按结果分类+脚本号查询（排除重试/补测） | ReportServiceImpl |
| getReTestReport(taskId) | 查询全部补测记录 | ReportServiceImpl |
| getScriptReportCount(taskId) | 统计脚本报告总数 | ReportServiceImpl |
| getNoPassCount(taskId) | 统计不通过数量（resultCategory > PASS） | ReportServiceImpl（失败通知阈值判断） |

## 代码引用

| 调用者 | 场景 |
|--------|------|
| `ReportServiceImpl` | 报告接收/删除/查询/更新主链路（insert 311、delete 256、getNoPassCount 354 失败通知、details 2219、baseList 3612/3626、getReTestReport 3641、update 4073/4417 等） |
| `TaskSummaryServiceImpl` | 读取报告列表做任务级汇总统计（list 76） |
| `GenerateReportServiceImpl` | 报告生成时读取报告详情（list 275/1692、details 1572） |
| `NoticeServiceImpl` | 邮件/通知内容组装（get 2423、baseList 2678/2789、list 3469/6430） |
| `ScriptSummaryServiceImpl` | 脚本汇总树维护时读取报告（list 599） |

## 相关文档

- [存储总览](../../web-pc处理服务/02-技术架构/00-存储总览.md)
- [PmTaskDetail](PmTaskDetail.md)
- [PmScriptSummary](PmScriptSummary.md)
- [PmTaskSummary](PmTaskSummary.md)
