# service-ErrorCauseTypeV3Api — 错误原因类型查询

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/realcfg/ErrorCauseTypeV3Api.java`（@Component）
> 类型：远端代理（→ RealCfg 服务）
> 转发方式：V3 REST 路径（`ServiceRemoteV3Api.remoteGet`，RestTemplate，基址 `ApiUtil.getServiceApiUrl("RealCfg")`）

## 方法列表

### 1. getErrorCause — 查询错误原因类型分页列表

```java
public PageResponseDTO<ErrorCauseTypeResponseDTO> getErrorCause(Integer eid, Integer projectId)
```

**用途**：拉取项目错误原因类型列表（固定 page=1、page_size=1000），报告生成时做错误归因展示。返回 null 抛 `apiInvalid`。

**转发目标**：`GET /v3/realcfg/error_cause_type/list?eid=&projectId=&page=1&page_size=1000`

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 id（直接拼接到 query，无 null 校验） |
| projectId | Integer | 否 | 项目 id |

**返回参数**：`PageResponseDTO<ErrorCauseTypeResponseDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| page | Integer | 当前页 |
| pageSize | Integer | 每页大小 |
| totalPage | Integer | 总页数 |
| totalRow | Integer | 总行数 |
| list | List&lt;ErrorCauseTypeResponseDTO&gt; | 错误原因类型列表（元素字段见 RealCfg 服务，代码未确认） |

**调用者**：
- `GenerateReportServiceImpl.java` /  /  / 

### 2. getErrorCauseMathRule — 查询错误类型匹配规则

```java
public PageResponseDTO<ErrorCauseMatchRuleResponseDTO> getErrorCauseMathRule(Integer projectId)
```

**用途**：查询项目的错误原因匹配规则（按规则自动归类失败原因）；异常时记录日志并返回空分页对象。

**转发目标**：`GET /v3/realcfg/error_cause_type/error_cause_match_rule?project_id=X`

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectId | Integer | 否 | 项目 id（直接拼接到 query，无 null 校验） |

**返回参数**：`PageResponseDTO<ErrorCauseMatchRuleResponseDTO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| page | Integer | 当前页 |
| pageSize | Integer | 每页大小 |
| totalPage | Integer | 总页数 |
| totalRow | Integer | 总行数 |
| list | List&lt;ErrorCauseMatchRuleResponseDTO&gt; | 匹配规则列表（元素字段见 RealCfg 服务，代码未确认） |

**调用者**：
- `ReportServiceImpl.java`

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealCfg 服务](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md)
- [service-ErrorCauseOperateLogV3Api](service-ErrorCauseOperateLogV3Api.md)
