# service-task-TestProcess — 执行机测试过程上报

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/service/task/TestProcess.java`（继承 `GenericBaseService`）
> 类型：ApiServlet 本地路由服务类
> 路由方式：action=task, op=TestProcess.<方法>
> 本地注入：`ITaskProcessService`

## 方法列表

### 1. report — 过程上报

```java
public String report(ApiRequest apirequest) throws Exception
```

**用途**：接收执行机上报的测试过程数据（开始、进度、结果等动作）。

**流程**：
1. 提取 `taskAction`（过程类型，对应 `TaskAction` 枚举）与 `content` JSON
2. taskAction 为空返回参数错误
3. 当 taskAction 为 `TaskAction.REPORT`（结果上报）时 content 必填，否则返回参数错误
4. `ITaskProcessService.report(taskAction, content)` 处理，返回结果码

**请求参数**（reqJson）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| taskAction | String | 是 | 过程类型（对应 `TaskAction` 枚举） |
| content | JSONObject | 是(条件) | 过程内容；仅当 taskAction=REPORT（结果上报）时必填 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | JSONObject | 业务数据 |
| data.result | Integer | 处理结果码 |

**涉及表**：

| 存储 | 表/集合 | 操作 |
|------|---------|------|
| MongoDB | PmTaskDetail / 过程记录 | 写（→ITaskProcessService.report） |

## 相关文档

- [00-分支索引](00-分支索引.md)
- [service-task-Task](service-task-Task.md)
- [service-task-TestResult](service-task-TestResult.md)（结果上报对等接口）
