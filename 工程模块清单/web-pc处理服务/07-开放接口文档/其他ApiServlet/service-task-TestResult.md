# service-task-TestResult — 执行机测试结果上报

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/service/task/TestResult.java`（继承 `GenericBaseService`）
> 类型：ApiServlet 本地路由服务类
> 路由方式：action=task, op=TestResult.<方法>
> 本地注入：`IReportService`

## 方法列表

### 1. report — 结果上报

```java
public String report(ApiRequest apirequest) throws Exception
```

**用途**：接收执行机上报的测试结果及结果文件，解析入库。

**流程**：
1. 提取 `taskResult` JSON 对象，为空返回参数错误
2. 提取 `resultFiles` JSON 对象（各结果文件，允许为空）
3. 校验 `taskResult.taskid` 非空
4. Gson 反序列化为 `TaskResult`，调用 `IReportService.reportResult(taskResult, resultFiles)` 解析入库
5. 返回 result=1/0

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskResult | JSONObject | 是 | 测试结果对象（反序列化为 `TaskResult`） |
| taskResult.taskid | String | 是 | 任务ID（taskResult 内必填） |
| resultFiles | JSONObject | 否 | 各结果文件 JSON（允许为空） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 1 成功 / 0 失败 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmReportDetail | 写（→IReportService.reportResult） |

## 相关文档

- [00-分支索引](00-分支索引.md)
- [service-task-Task](service-task-Task.md)
- [service-task-TestProcess](service-task-TestProcess.md)（过程上报对等接口）
