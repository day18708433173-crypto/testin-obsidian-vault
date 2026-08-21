# service-quartz-Report — 定时任务场景获取任务报告 url

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/service/quartz/Report.java`（无父类，SpringContextUtil 手动取 Bean）
> 类型：ApiServlet 本地路由服务类
> 路由方式：action=quartz, op=Report.<方法>
> 本地注入：`ReportApi`

## 方法列表

### 1. url — 获取任务报告 url

```java
public String url(ApiRequest apiRequest) throws GeneralException
```

**用途**：定时任务场景下按业务类型获取 app/web 任务的报告地址。

**流程**：
1. 校验 taskid、projectid 必填
2. 读取 `businessType`（默认 "2" 即 web）
3. businessType=app → `ReportApi.getAppReportUrl(taskId, projectId)`；businessType=web → `ReportApi.getWebReportUrl(taskId, projectId)`
4. url 为空抛 `GeneralException(noneData, "报告url为空")`，否则返回 url

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskid | String | 是 | 任务ID |
| projectid | Integer | 是 | 项目组ID |
| businessType | String | 否 | 业务类型，默认 "2"（web）；app 取值 app 枚举 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | String | 任务报告 url |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| Remote（经 ReportApi） | 报告地址查询 | 读 |

## 相关文档

- [00-分支索引](00-分支索引.md)
- [service-quartz-Quartz](service-quartz-Quartz.md)
- [service-report-Report](service-report-Report.md)（报告页 Report.url 为本机拼接版本）
