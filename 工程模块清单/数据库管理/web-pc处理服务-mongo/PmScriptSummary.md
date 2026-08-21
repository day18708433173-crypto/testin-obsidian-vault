# MongoDB 集合 — PmScriptSummary

> 数据库：pmweb_db 或 pmpc_db | 集合：`pmwebScriptSummary_00~99` / `PmpcScriptSummary_00~99`
> DAO：`PmScriptSummaryDAOImpl` (implements `IPmScriptSummaryDAO`)
> 实体：`cn.testin.realweb.pojo.mongo.PmScriptSummary`

## 分片规则

`PmScriptSummary.table(taskid)` → `BasePojo.getTable(taskid, "pmwebScriptSummary" 或 "PmpcScriptSummary")`：

- taskId 末 2 位数字 → `_00` ~ `_99`
- taskId 以 `pt` 开头 → pmpc_db（`PmpcScriptSummary_XX`），否则 → pmweb_db（`pmwebScriptSummary_XX`）

详见 [存储总览](../../web-pc处理服务/02-技术架构/00-存储总览.md)。

## 用途

脚本汇总树：每个脚本（含嵌套子脚本）一条文档，聚合步骤执行列表、性能统计项、报告关联（子子任务id列表）、结果分类汇总与补测信息。是报告页"按脚本视角"展示的数据源，通过 `reportDetails`/`subsubtaskids` 反查 [PmReportDetail](PmReportDetail.md) 中的报告详情。

静态方法 `generateScriptSign(scriptNo, order)`：生成 `scriptNo + "_" + order` 编号（同 scriptNo 多次提测的区分标识）。

## 主要字段

| 字段 | 类型 | 说明 |
|------|------|------|
| taskid | String | 任务ID（分片键、查询主条件） |
| projectid | Integer | 项目ID |
| scriptid | Integer | 脚本ID |
| scriptNo | Integer | 脚本号 |
| scriptSign | String | 脚本编号（scriptNo + "_" + 同 scriptNo 提测次序） |
| scripts | List\<ScriptInfo\> | 脚本列表（含自身主脚本及嵌套关联脚本） |
| steps | List\<StepInfo\> | 步骤执行列表（步骤信息、失败设备数、图片数量） |
| capabilityItems | String[] | 性能统计项目汇总 |
| reportDetails | List\<ReportDetail\> | 报告关联信息（存子子任务id列表，可反查报告详情） |
| orderNum | Integer | 提测时的脚本顺序（update 条件之一） |
| categorySummarys | List\<ResultCategorySummary\> | 结果分类列表信息 |
| build | Integer | 脚本版本号 |
| retestSequence | Integer | 补测顺序 |
| retestInfos | List\<RetestInfo\> | 补测信息 |
| retryNum | Integer | 出错重试序号 |
| uuid | String | uuid（findScriptSummaryByUuid/updateSubSubTaskidsByUUID 条件） |
| subsubtaskids | List\<String\> | 子子任务id列表（补测信息） |
| status / updatetime | Integer/Long | 继承 BasePojo |

子结构位于 `cn.testin.realweb.pojo.mongo.scriptSummary` 包：`ScriptInfo`、`StepInfo`、`ReportDetail`、`ResultCategorySummary`、`RetestInfo`。

## 核心操作

| DAO 方法 | 操作 | 调用者 |
|----------|------|--------|
| batchInsert(list) | 批量插入 | ScriptSummaryServiceImpl（汇总树初始化/补测插入） |
| list(taskid, conditionMap, keywords) | 条件列表（orderNum/scriptid/scriptNo/reportDetails.subsubtaskid，按 orderNum 升序） | ScriptSummaryServiceImpl, ReportServiceImpl |
| listByOrderNum(taskid, orderNums, keywords) | 按 orderNum 批量查询 | ScriptSummaryServiceImpl, ReportServiceImpl |
| update(scriptSummary) | 按 taskid+scriptNo/orderNum/scriptid 更新 scripts/steps/reportDetails/categorySummarys | ScriptSummaryServiceImpl（汇总树维护） |
| updateByRetest(list) | 批量按 taskid+scriptNo/orderNum/scriptid 更新 retestInfos/retestSequence/scripts/steps | ScriptSummaryServiceImpl（补测） |
| getList(taskid, subsubtaskids) | 按 subsubtaskids 反查所属脚本汇总 | ScriptSummaryServiceImpl |
| getBySubSubTaskId(taskid, subsubtaskid, keywords) | 按单个子子任务id查一条 | ReportServiceImpl |
| findScriptSummaryByUuid(taskid, uuid) | 按 uuid 查单条 | ReportServiceImpl, TaskProcessServiceImpl |
| updateSubSubTaskidsByUUID(updateScriptSummary) | 按 taskid+uuid 更新 subsubtaskids | TaskProcessServiceImpl（重测关联修复） |
| getListByUUID(taskid, uuidList, keywords) | 按 uuid 列表批量查询 | ScriptSummaryServiceImpl |

## 代码引用

| 调用者 | 场景 |
|--------|------|
| `ScriptSummaryServiceImpl` | 汇总树生成/维护主链路：list（84/580/1119）、batchInsert（216/1489/1883）、update（855/1327）、updateByRetest（1482/1875）、getList（1543） |
| `ReportServiceImpl` | 上报结果定位所属脚本：findScriptSummaryByUuid（229）、getBySubSubTaskId（236）、listByOrderNum（243）；报告页按脚本读取 list（1231/2267/2720） |
| `TaskProcessServiceImpl` | 重测时按 uuid 找脚本汇总并更新 subsubtaskids（findScriptSummaryByUuid 325、updateSubSubTaskidsByUUID 339）——"修复重测找不到脚本"相关链路 |
| `TaskServiceImpl` | 经 `IScriptSummaryService` 间接调用（约 1002 行） |
| `NoticeServiceImpl` | 经 `IScriptSummaryService` 读取汇总树组装通知（383/1636/5312 等） |

## 相关文档

- [存储总览](../../web-pc处理服务/02-技术架构/00-存储总览.md)
- [PmTaskDetail](PmTaskDetail.md)
- [PmReportDetail](PmReportDetail.md)
- [PmScriptRunInfo](PmScriptRunInfo.md)
