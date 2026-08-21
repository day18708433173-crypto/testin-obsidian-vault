# service-ErrorCauseOperateLogV3Api — 错误原因操作日志写入

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/realcfg/ErrorCauseOperateLogV3Api.java`（@Component）
> 类型：远端代理（→ RealCfg 服务）
> 转发方式：V3 REST 路径（`ServiceRemoteV3Api.remote`，POST，基址 `ApiUtil.getServiceApiUrl("RealCfg")`）

## 方法列表

### 1. insertOperateLog — 写入错误原因操作日志

```java
public Integer insertOperateLog(ErrorCauseOperateLogRequestDTO requestDTO) throws GeneralException
```

**用途**：用户人工修改脚本失败原因分类时，向 RealCfg 写入一条操作日志（审计用）。返回写入结果（Integer）。

**转发目标**：`POST /v3/realcfg/error_cause_operate_log/save_log`（body 为 ErrorCauseOperateLogRequestDTO）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| requestDTO | ErrorCauseOperateLogRequestDTO | 是 | 错误原因操作日志请求体（操作人、脚本、修改前后原因等） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Integer | 写入结果（`ResultResponseDTO<Integer>.result`） |

**调用者**：
- `ScriptRunInfoServiceImpl.java`
- `ReportServiceImpl.java` / 

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealCfg 服务](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md)
- [service-ErrorCauseTypeV3Api](service-ErrorCauseTypeV3Api.md)
