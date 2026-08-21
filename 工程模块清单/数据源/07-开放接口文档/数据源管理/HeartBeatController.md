# HeartBeatController — 心跳检测控制器

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/controller/HeartBeatController.java`
> 类级路由：`/datasource`（完整前缀 `/openapi/v3/datasource`）
> 业务：服务健康检查。

## 接口列表总表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|------|------|--------|------|
| 1 | GET | `/v3/datasource/heartbeat/check` | check | 心跳检测（执行 SELECT 1） |

---

## 1. GET /v3/datasource/heartbeat/check — 心跳检测

### 入口

`HeartBeatController.check()`

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| — | — | — | 无参数 |

### 返回参数

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data | Object | 空对象（成功时返回 `{}`） |

### 实现意图

执行 `heartBeatCheckMapper.heartbeatCheck()` → `SHOW STATUS` 确认 MySQL 连接正常。成功返回 `ResponseResult.success({})`；数据库不可达时抛出 `HealthyCheckGenericException`，被 `@ExceptionHandler` 拦截返回 HTTP 500。

### 代码摘录

```java
@RequestMapping(value = "/heartbeat/check", method = RequestMethod.GET)
@ResponseBody
public ResponseResult check() {
    heartBeatCheckMapper.heartbeatCheck();
    return ResponseResult.success(new HashMap<>());
}

@ExceptionHandler(HealthyCheckGenericException.class)
public ResponseEntity<ResponseResult> handleHealthyCheckGenericException(
        HealthyCheckGenericException e) {
    return new ResponseEntity<>(
        ResponseResult.error(e.getErrCode(), e.getErrMsg()),
        HttpStatus.INTERNAL_SERVER_ERROR);
}
```
