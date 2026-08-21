# service-ActionLogController — 通用活动日志接口

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/common/ActionLogController.java`
> 类：`cn.testin.service.common.ActionLogController`（Spring MVC `@RestController`，同时兼容 ApiServlet 反射调用）
> 双入口方式：
> - **ApiServlet 入口**：`action=common`，`op=ActionLogController.getActionLog` 反射调用（GET 风格，返回 JSON 字符串）
> - **action**: `common`（对应包 `cn.testin.service.common`）
> - **入口格式**：`{"op": "ActionLogController.getActionLog", "action": "common", "data": {...}}`
> - **Spring MVC 入口**：`@RequestMapping("/core/action/")` 下的两个接口
> 依赖：`IActionLogService`（通过 `SpringUtil.getBean` 获取）
> 业务：活动记录的创建/导出/复制等操作日志的查询（分页列表、按 requestId 查单条）。
> 涉及表：`db_common.action_log`

## 方法列表总表

| # | 方法 | HTTP 路由 | 说明 | 返回类型 |
|---|---|---|---|---|
| 1 | getActionLogsByPage | `POST /core/action/records` | 分页查询活动记录列表 | `ResponseResult<BasePageListResponseDTO<ActionLogVO>>` |
| 2 | getActionLog | `GET /core/action/record` | 按 requestId 查单条活动记录 | `String`（JSON 字符串） |

> 注意：两个方法走不同风格。
> - `getActionLogsByPage` 是标准 Spring MVC Controller，参数经 `@RequestBody @Valid` 自动绑定与校验。
> - `getActionLog` 同时兼容 ApiServlet 反射调用（`action=common` + `op=ActionLogController.getActionLog`），参数从 `ApiRequest.reqjson` 手工提取。

---

## 双入口分发机制

### 入口 A：Spring MVC

- 路由前缀：`/core/action/`
- `POST /core/action/records` → `getActionLogsByPage(ActionLogQueryCondition)`
- `GET /core/action/record` → `getActionLog(ApiRequest)`

### 入口 B：ApiServlet（反射派发）

- 入口：`/*`
- `action` 参数 = `common`（定位到 `cn.testin.service.common` 子包）
- `op` 参数 = `ActionLogController.getActionLog`
- 此时仅 `getActionLog` 方法可被访问（方法签名 `String xxx(ApiRequest)` 匹配反射约定）
- `getActionLogsByPage` 方法签名不同（参数为 `ActionLogQueryCondition`，返回 `ResponseResult`），**不可**通过 ApiServlet 反射调用

---

## 1. getActionLogsByPage — 分页查询活动记录（Spring MVC 专用）

### 请求参数（`@RequestBody @Valid ActionLogQueryCondition`）

| 字段 | 必填 | 说明 |
|---|---|---|
| page | 是 | 当前页码 |
| pageSize | 是 | 每页大小 |
| moduleId | 是 | 模块ID |
| actionTypeList | 否 | 活动类型列表（`List<Integer>`），用于按类型筛选 |

### 响应结构

`ResponseResult<BasePageListResponseDTO<ActionLogVO>>` 标准分页结构：
- `code` / `msg`：状态码 / 消息
- `data.list`：`List<ActionLogVO>` 活动记录列表
- `data.page` / `data.pageSize` / `data.total`：分页信息

### 代码摘录

```java
@PostMapping("records")
public ResponseResult<BasePageListResponseDTO<ActionLogVO>> getActionLogsByPage(
        @RequestBody @Valid ActionLogQueryCondition condition) throws Exception {
    BasePageListResponseDTO<ActionLogVO> result = actionLogService.getActionLogsByPage(condition);
    return ResponseResult.success(result);
}
```

---

## 2. op=ActionLogController.getActionLog — 按 requestId 查单条活动记录（ApiServlet + Spring MVC 双入口）

### 请求格式（ApiServlet入口）
{"op": "ActionLogController.getActionLog", "action": "common", "data": {"requestId": "..."}}

### 请求参数

| 字段 | 必填 | 说明 |
|---|---|---|
| requestId | 是 | 活动记录请求ID（不存在时抛出 `CommonCode.paraInvalid` 异常） |

> ApiServlet 入口时从 `ApiRequest.reqjson` 取；Spring MVC GET 入口时从 `ApiRequest.reqjson` 取（框架自动绑定）。

### 代码摘录

```java
@GetMapping("record")
public String getActionLog(ApiRequest apiRequest) throws GeneralException {
    JSONObject reqJson = apiRequest.getReqjson();
    if (!reqJson.has("requestId")) {
        throw new GeneralException(CommonCode.paraInvalid.getValue(), "invalid requestId");
    }
    String requestId = reqJson.optString("requestId");
    JSONObject jObj = ApiUtil.getJSONobj(apiRequest, CommonCode.success.getValue(),
            CommonCode.success.getDescr());
    ActionLog actionLog = actionLogService.getActionLogByRequestId(requestId);
    JSONObject resultJson = new JSONObject();
    if (actionLog != null) {
        ActionLogVO actionLogVO = new ActionLogVO();
        BeanUtil.copyProperties(actionLog, actionLogVO);
        resultJson = new JSONObject(actionLogVO);
    }
    jObj.put(ApiResponse.RES_DATA, resultJson);
    return jObj.toString();
}
```

### 响应结构

`data` = `ActionLogVO` 的 JSONObject（查不到时为空 JSONObject `{}`）。

`ActionLogVO` 字段（来自表 `db_common.action_log`）：

| 字段 | 类型 | 说明 |
|---|---|---|
| id | Integer | 主键ID |
| projectId | Integer | 项目ID |
| moduleId | Integer | 模块ID |
| successCount | Integer | 成功数量（如脚本复制成功数） |
| failCount | Integer | 失败数量 |
| status | Byte | 状态 |
| failCause | String | 失败原因 |
| actionType | Integer | 活动类型 |
| requestId | String | 请求ID（查询的主键条件） |
| result | String | 活动结果（如文件导出地址） |
| createUserId | Integer | 创建人ID |
| createUserName | String | 创建人名称 |
| createTime | Long | 创建时间 |
| updateTime | Long | 更新时间 |
| desc | String | 活动描述 |
| ext | String | 扩展字段（fail_cause 的 JSON 字符串） |

### 涉及的数据库操作

- `getActionLogsByPage`：`actionLogService.getActionLogsByPage(condition)` — 表 `db_common.action_log`，分页条件查询
- `getActionLog`：`actionLogService.getActionLogByRequestId(requestId)` — 表 `db_common.action_log`，按 `request_id` 查单条

---

## 备注

- 本类与同目录下其他 `GenericBaseService` 子类不同，它标注了 `@RestController` 和 `@RequestMapping`，是 Spring MVC 管理的 Bean（而非单纯靠 ApiServlet 反射实例化）。这使其同时拥有两套入口。
- `getActionLog` 方法返回 `String`（ApiServlet 风格），但标注了 `@GetMapping("record")`，因此也可直接被 Spring MVC 路由访问，路径为 `GET /core/action/record?reqjson={"requestId":"xxx"}`（注：GET 请求下 reqjson 需经过框架的 query string 到 JSON 的绑定）。
- `ActionLogVO` 与 `ActionLog` 字段一一对应，通过 `BeanUtil.copyProperties` 转换。VO 不含 `eid` 字段（`ActionLog` 有此字段但不对外暴露）。
- `IActionLogService` 通过 `SpringUtil.getBean` 获取（非 `@Autowired`），写法与其他 GenericBaseService 子类一致。

相关文档：[00-分支索引](00-分支索引.md)
