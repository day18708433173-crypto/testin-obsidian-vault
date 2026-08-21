# service-TestPlanApi — 测试计划执行回调

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/testplan/TestPlanApi.java`（SpringHelper Bean `TestPlanApi`）
> 类型：远端代理（→ RealLogfile 服务）
> 转发方式：HttpClient 直连 REST（POST JSON 到 `service.api.properties` 中 `RealLogfile` 地址 + `/v3/test_plan/execute_record_tasks/callback`）

## 方法列表

### 1. report — 通知测试计划模块

```java
public boolean report(TaskExecuteCallbackRequestDTO callbackRequestDTO) throws GeneralException
```

**用途**：任务执行结束后回调 RealLogfile 测试计划模块，更新计划执行记录下的任务状态；返回 result==1 视为成功。非 200 或异常记日志返回 false。

**转发目标**：`POST {RealLogfile}/v3/test_plan/execute_record_tasks/callback`（body 为 TaskExecuteCallbackRequestDTO JSON）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| callbackRequestDTO | TaskExecuteCallbackRequestDTO | 是 | 回调内容（执行记录、任务 id、结果状态等），null 直接返回 false |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Boolean | 远端 result==1 为 true，否则 false |

**调用者**：
- `ReportServiceImpl.java` / （报告完成后回调）
- `NoticeServiceImpl.java` /  /  /  / （通知流程中回调）

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealLogfile 服务](../../../平台基础功能服务/00-首页.md)
- [service-DirQuartzApi](service-DirQuartzApi.md)
