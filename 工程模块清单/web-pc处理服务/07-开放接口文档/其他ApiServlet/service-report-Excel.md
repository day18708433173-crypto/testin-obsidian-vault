# service-report-Excel — 报告 Excel 异步生成与获取

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/service/report/Excel.java`（继承 `GenericBaseService`）
> 类型：ApiServlet 本地路由服务类
> 路由方式：action=report, op=Excel.<方法>
> 本地注入：`ITaskDetailService`, `IMqInfoNoticeDAO`, `IGenerateReportService`, `ReportApi`

## 方法列表

### 1. reportExcel — 获取报告 Excel（轮询式）

```java
public String reportExcel(ApiRequest apiRequest) throws Exception
```

**用途**：获取报告 Excel 下载地址；未生成时投递 MQ 异步生成，前端轮询。result 语义：1 成功 / 0 生成中继续轮询 / -1 生成失败 / -2 待执行不可生成。

**流程**：
1. skey 经 `ReportApi.getTaskIdByShareId` 换算 taskid；提取 eid/userid
2. 父类 `taskInfoSupplement` 校验并补充任务信息（taskid/skey 至少其一）
3. `checkSummaryNotice`：查 MQ 通知表 `REPORT_STAT` 待处理记录，若统计未完成（status≠0 且未达最大执行次数）抛 `GeneralException("任务正在统计中")`
4. `ITaskDetailService.get(taskid, null)` 读 PmTaskDetail，为空返回 noneData
5. 解析 `taskDetail.content` 中的 `excel` 子对象：
   - 不存在 → `addExcel` 投递生成
   - 存在但缺 updatetime/url 且 status≠0（非生成中）→ `addExcel` 重新投递
   - `excel.updatetime >= taskDetail.finishTime`（任务结束后已生成最新版）→ `returnExceUrl` 返回 status + excelUrl
   - 否则 → `addExcel` 返回等待
6. `addExcel` 内部：先查 `REPORT_GENERATE` 待处理通知，已存在直接返回 result=0；否则 `IGenerateReportService.addReportNotice(taskid)` 投递，成功 0 / 失败 -1

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是(与skey二选一) | 任务ID |
| skey | String | 否 | 分享key，与 taskid 至少一个 |
| eid | Integer | 否 | 企业ID |
| userid | Integer | 否 | 用户ID |
| userprojectids | JSONArray | 否 | 用户项目ID集合（归属校验） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 1 成功 / 0 生成中（继续轮询）/ -1 生成失败 / -2 待执行不可生成 |
| data.excelUrl | String | Excel 下载地址（仅生成完成后返回） |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmTaskDetail | 读 |
| MySQL（common） | mq_info_notice | 读（REPORT_STAT / REPORT_GENERATE 待处理查询） |
| MQ | 报告生成通知 | 写（→IGenerateReportService.addReportNotice） |

## 相关文档

- [00-分支索引](00-分支索引.md)
- [service-report-Report](service-report-Report.md)
- [service-report-Pdf](service-report-Pdf.md)
