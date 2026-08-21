# service-PreciseTestApi — CICC 精准测试覆盖率

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/cicc/PreciseTestApi.java`（SpringHelper Bean `cicc.PreciseTestApi`）
> 类型：远端代理（→ CICC 精准测试服务）
> 转发方式：直连 HTTP（`HttpPoster.postJSONWithRes`，基址 `Config.OEM_CICC_PRECISE_TEST_URL`，token = DES 加密当前时间戳，DesKey `&cTp1aT*`）

## 方法列表

### 1. generateCoverData — 生成覆盖数据报告

```java
public String generateCoverData(String sessionId, Integer type) throws Exception
```

**用途**：任务结束后通知 CICC 按 session 统计覆盖率，返回 tid；type 区分来源（301 定时任务 / 302 手工任务 / 303 流水线），realtime 固定 true。

**转发目标**：`POST /covs/meter/covs/stat`

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| sessionId | String | 是 | 会话 id（覆盖统计维度） |
| type | Integer | 是 | 来源类型（301 定时任务 / 302 手工任务 / 303 流水线） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| tid | String | 覆盖率统计任务 id（响应 result.tid） |

**调用者**：`TaskProcessServiceImpl.java`、`ReportServiceImpl.java` / 

### 2. clearCoverData — 清空覆盖数据

```java
public String clearCoverData(String sessionId) throws Exception
```

**用途**：任务开始前清空该 session 的历史覆盖数据，避免统计串扰。

**转发目标**：`POST /covs/meter/covs/reset`

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| sessionId | String | 是 | 会话 id |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | String | 原始响应字符串（res） |

**调用者**：`TaskProcessServiceImpl.java` / 、`ReportServiceImpl.java`

### 3. saveCoverData — 保存覆盖数据

```java
public String saveCoverData(String tid) throws Exception
```

**用途**：把 generateCoverData 产出的覆盖率结果持久化到 minio，返回报告查看地址（`Config.OEM_CICC_PRECISE_REPORT_URL + tid`）。

**转发目标**：`POST /covs/meter/covs/minio/save`

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| tid | String | 是 | 覆盖率统计任务 id |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | String | 报告查看地址（`Config.OEM_CICC_PRECISE_REPORT_URL + tid`） |

**调用者**：`TaskProcessServiceImpl.java`、`ReportServiceImpl.java` / 

## 相关文档

- [00-分支索引](00-分支索引.md)
- CICC 服务
- [service-EnvConfigApi](service-EnvConfigApi.md)（精准测试开关读取）
