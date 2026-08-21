# service-AuditData — 审计数据采集接口

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/audit/AuditData.java`
> 类：`cn.testin.service.audit.AuditData extends GenericBaseService`（非 Spring MVC Controller，无注解路由）
> 入口方式：ApiServlet 入口，`action=audit`，`op=AuditData.collect` 反射调用
> - **action**: `audit`（对应包 `cn.testin.service.audit`）
> - **入口格式**：`{"op": "AuditData.collect", "action": "audit", "data": {...}}`
> 依赖：`IAuditInfoService`（Spring Bean，继承自 `GenericBaseService`）
> 业务：接收并持久化前端/其他模块上报的 API 调用审计记录（请求/响应/耗时/操作人）。
> 涉及表：`db_user.audit_info`

## 方法列表总表

| # | 方法 | 说明 | 主要依赖 |
|---|---|---|---|
| 1 | collect | 采集并存储一条审计数据（请求+响应+耗时+操作人） | iAuditInfoService.insert |

统一返回：JSON 字符串，结构 `{ code, msg, data }`；`data` 内含 `result`（Boolean）。

---

## 分发机制

- 入口：`/*`（ApiServlet）
- `action` 参数 = `audit`（定位到 `cn.testin.service.audit` 子包）
- `op` 参数 = `AuditData.collect`
- 请求体中 `reqjson` 为业务 JSON

---

## 1. op=AuditData.collect — 采集审计数据

### 请求格式
{"op": "AuditData.collect", "action": "audit", "data": {"moduleId": ..., "name": "...", "totalTime": ..., "reqData": {...}, "respData": {...}, "onlineUserInfoJson": {...}}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| moduleId | 否 | 模块ID |
| moduleName | 否 | 模块名称 |
| name | 否 | 操作名称/接口名称 |
| totalTime | 否 | 执行耗时（毫秒，Long） |
| action | 否 | action 参数值 |
| op | 否 | op 参数值（具体方法名） |
| reqData | 否 | 请求数据（JSONObject，落库时 toString） |
| respData | 否 | 响应数据（JSONObject，落库时 toString） |
| code | 否 | 返回码 |
| msg | 否 | 返回信息 |
| onlineUserInfoJson | 否 | 在线用户信息 JSON，内含 `eid` / `projectid` / `userid` |

### 代码摘录

```java
public String collect(ApiRequest request) throws Exception {
    // ... 从 reqjson 和 onlineUserInfoJson 提取各字段 ...
    DbAuditInfo dbAuditInfo = new DbAuditInfo();
    dbAuditInfo.setUserId(userId);
    dbAuditInfo.setName(name);
    dbAuditInfo.setEid(eid);
    dbAuditInfo.setProjectId(projectId);
    dbAuditInfo.setModuleId(moduleId);
    dbAuditInfo.setModelName(moduleName);
    dbAuditInfo.setExccuteTime(totalTime);       // 注意: 拼写为 executeTime 的变体
    dbAuditInfo.setAction(action);
    dbAuditInfo.setOp(op);
    dbAuditInfo.setReqData(reqData.toString());
    dbAuditInfo.setRespData(respData.toString());
    dbAuditInfo.setResCode(code);
    dbAuditInfo.setResMsg(msg);
    Integer result = iAuditInfoService.insert(dbAuditInfo);
    // ... 组装返回 ...
    dataMap.put(ApiResponse.RES_RESULT, result > 0);
    // ...
}
```

### 响应结构

`data.result` = Boolean，`true` 表示插入成功（影响行数 > 0），`false` 表示失败。

### 实现意图

通用的 API 调用审计记录采集入口。前端或其他微服务在完成一次 API 调用后，将请求参数、响应结果、耗时、操作人等信息通过此接口上报存储到审计表，用于后续审计追溯和问题排查。

### 涉及的数据库操作

`iAuditInfoService.insert(dbAuditInfo)` — 表 `db_user.audit_info`，字段包括：

| 数据库字段 | Java 属性 | 说明 |
|---|---|---|
| id | id | 主键自增 |
| name | name | 操作名称/接口名 |
| eid | eid | 企业ID |
| projectid | projectId | 项目ID |
| userid | userId | 用户ID |
| module_id | moduleId | 模块ID |
| model_name | modelName | 模块名称 |
| exccutetime | exccuteTime | 执行耗时（毫秒） |
| res_code | resCode | 返回码 |
| res_msg | resMsg | 返回信息 |
| action | action | action 参数值 |
| op | op | op 参数值 |
| req_data | reqData | 请求JSON（toString存储） |
| resp_data | respData | 响应JSON（toString存储） |
| status | status | 状态（1有效/0无效） |
| createtime | createTime | 创建时间 |
| updatetime | updateTime | 更新时间 |

---

## 备注

- 操作人信息（userId/eid/projectId）从 `onlineUserInfoJson` 子对象中提取，而非直接从 `reqjson` 顶层取——这意味着调用方需要将当前登录用户信息作为嵌套 JSON 传入。
- `exccutetime` 字段名存在拼写错误（应为 executeTime），属历史遗留命名。
- `reqData` 和 `respData` 以 `toString()` 方式存储完整的 JSON 字符串，可能存在敏感数据泄露风险，建议审计数据查询接口做权限控制。
- 该方法没有对必填参数做强制校验，所有字段均可为空（full-optional），尽可能不漏掉审计记录。

相关文档：[00-分支索引](00-分支索引.md)
