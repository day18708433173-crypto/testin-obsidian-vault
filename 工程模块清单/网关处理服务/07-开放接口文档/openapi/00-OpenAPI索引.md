---
module: proxy-openapi
type: 开放接口文档
source: db_mcfg view_role_api + 各服务代码
---

# OpenAPI 接口文档（Swagger 风格总索引）

> 本目录是网关（proxy-openapi）代理的全部「已归属知识库」业务接口的 Swagger 风格汇总，逐接口给出**转发模式、鉴权、请求参数（含必填标记）与返回参数**。字段信息来自各服务代码分析，非官方文档中心的泛化说明。
>
> 数据来源：[01-API路由映射表](../01-API路由映射表.md)（db_mcfg 库 view_role_api 实时数据）+ 各服务本地代码仓（分支 `syy.release.z7.8.1.0`）。

## 全局鉴权

所有外部请求经网关（ApiProxyServlet / ApiV2ProxyServlet / ApiV3ProxyServlet）需通过认证链（见 [00-模块索引](../00-模块索引.md)）：

| 参数 | 必填 | 说明 |
|---|---|---|
| apikey | 是 | 应用身份标识（McfgApp） |
| mkey | 是 | 目标模块标识（McfgModule） |
| action / op | V1/V2 必填 | 接口包名 / 方法名 |
| sid | needLogin=1 时必填 | 会话 token（或 Cookie authtoken_pro） |
| timestamp | AppConfig.checkTimestamp=1 时 | 时间戳 |
| sig | AppConfig.checkSig=1 时 | MD5 签名（key 字母序递归拼接 + secretKey） |

## 转发模式

| 标记 | 模式 | pass_through_type | 判定 |
|---|---|---|---|
| 🟢 | V1 原生 | 0 | api_action 为短格式（`app.Task.add`） |
| 🔵 | V3 透传 | 1 | api_action 为 URL 格式 |
| 🟡 | V3→V1 转换 | 0 | URL 格式 + special_api_action/op 非空 |

## 通用返回结构

- **V1（ApiServlet，action/op）**：`{"code": 0, "message": "...", "data": {...}}`，`code=0` 表示成功。
- **V3（Spring MVC）**：统一 `BaseResponseDTO` / `BaseResultStrResponseDTO`，形如 `{"code":0,"message":"...","data":{...}}`；分页接口 `data` 内含 `total`/`list` 等字段，以各接口文档为准。

## 服务索引

| # | 服务 | mkey / 模块 | 目标服务 | OpenAPI 明细 |
|---|---|---|---|---|
| 1 | app处理服务（real-test） | realtest | testin-aio-real-test:8080 | [01-app处理服务](openapi/01-app处理服务.md) |
| 2 | web-pc处理服务（real-web） | realweb / realpc / common | testin-aio-real-web:8080 | [02-web-pc处理服务](openapi/02-web-pc处理服务.md) |
| 3 | 任务管理服务（real-task） | real_task | testin-aio-real-task:8080 | [03-任务管理服务](openapi/03-任务管理服务.md) |
| 4 | 任务调度服务（real-scheduling） | RealScheduling | testin-aio-real-scheduling:8080 | [04-任务调度服务](openapi/04-任务调度服务.md) |
| 5 | 设备控制中心（real-controlcenter） | controlcenter / device / UcomDeivce / device_monitor | testin-aio-real-controlcenter:8080 | [05-设备控制中心](openapi/05-设备控制中心.md) |
| 6 | 平台基础功能服务（testin-core） | core / user / notice / project / test_plan / test_case / task / realportal / defect / logfile / log | testin-aio-testin-core:8080 | [06-平台基础功能服务](openapi/06-平台基础功能服务.md) |
| 7 | 平台配置（real-cfg） | realcfg / env | testin-aio-real-cfg:8080 | [07-平台配置](openapi/07-平台配置.md) |
| 8 | 脚本服务（filesystem） | script / apppackage / suite / file | testin-aio-filesystem:8080 | [08-脚本服务](openapi/08-脚本服务.md) |
| 9 | 文件管理服务（fileupload） | fs / file_system | testin-aio-fileupload:8080/openapi | [09-文件管理服务](openapi/09-文件管理服务.md) |
| 10 | 数据源（datasource-manage） | datasource / datasource-manage | testin-aio-datasource-manage:8080/openapi | [10-数据源](openapi/10-数据源.md) |

## 未归属知识库的服务（仅路由记录，无代码仓深挖）

| mkey | 目标服务 | API 数 | 主要转发模式 | 说明 |
|---|---|---|---|---|
| admin | 管理中心 | 50 | 🔵 + 🟡 混合 | 运维后台：设备资产管理/天统计/用户管理/日志查询 |
| analysis | 分析服务 | — | — | testin-aio-real-analysis:8080 |
| devops | 环境运维 | 49 | 🟢 | testin-aio-devops:8080 |
| testin-plan | 测试计划(独立) | 36 | 🟢 | testin-aio-test-plan:8080/openapi |
| testin-third | 第三方缺陷 | 21 | 🟢 | testin-aio-testin-third:8080/openapi |
| prodatatrans | 数据传输 | 32 | 🟢 | testin-aio-datatrans:8080 |
| realcross | 跨端 | 10 | 🟢 | testin-aio-real-cross:8080 |
| remote_report | 真机调试 | 25 | 🟢 | testin-aio-assistant-web:8080/openapi |
| statis | 统计 | — | — | testin-aio-statis:8080 |
| TestManageAdapt | 测管提测 | 1 | 🟢 | testin-aio-testmanageadapt:8080/openapi |
| opsapi | 运维 WebShell | 1 | 🟢 | testin-aio-serverops:8080 |
| report | 报表 | 2 | 🔵 | testin-aio-testin-core:8080 |
| logfile | 日志服务 | 8 | 🟢 | testin-aio-testin-core:8080 |
