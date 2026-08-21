---
branch: syy.release.z7.8.1.0
module: real-scheduling
type: 接口文档
entry: WebMvc
---

# GlobalExceptionHandler

全局异常处理器（`@RestControllerAdvice`），统一处理 任务调度服务 模块 MVC 层的异常响应。

## 异常处理

### handleGeneralException (`@ExceptionHandler(GeneralException.class)`)

- **入口**：`cn.testin.config.exception.GlobalExceptionHandler#handleException(GeneralException)`
- **实现意图**：捕获业务通用异常，从异常对象中提取错误码和错误消息，封装为 `ResponseResult` 返回。
- **返回参数**：

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 错误码（exception.getCode()） |
| msg | String | 错误信息（exception.getMsg()） |
| data | JSONObject | 空对象 `{}` |
- **关键代码摘录**：
```java
@ExceptionHandler(GeneralException.class)
public ResponseResult<Object> handleException(GeneralException exception) {
    Logit.errorLog("请求处理发生错误:" + exception.getMsg(), exception);
    ResponseResult<Object> result = new ResponseResult<>(exception.getCode(), exception.getMsg());
    result.setData(new HashMap<>());
    return result;
}
```

---

### handleMethodArgumentNotValidException (`@ExceptionHandler(MethodArgumentNotValidException.class)`)

- **入口**：`cn.testin.config.exception.GlobalExceptionHandler#handleException(MethodArgumentNotValidException)`
- **实现意图**：捕获 Spring Validation 参数校验失败异常，提取字段级别的错误消息返回。
- **返回参数**：

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 错误码（CommonCode.paraInvalid） |
| msg | String | 字段校验错误信息（无字段错误时默认「参数错误」） |
| data | JSONObject | 空对象 `{}` |
- **关键代码摘录**：
```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseResult<Object> handleException(MethodArgumentNotValidException exception) {
    Logit.errorLog("请求参数不正确:" + exception.getMessage());
    String defaultMessage = exception.getBindingResult().getFieldError() != null
        ? exception.getBindingResult().getFieldError().getDefaultMessage() : "参数错误";
    ResponseResult<Object> result = new ResponseResult<>(CommonCode.paraInvalid.getValue(), defaultMessage);
    result.setData(new HashMap<>());
    return result;
}
```

---

### handleException (`@ExceptionHandler(Exception.class)`)

- **入口**：`cn.testin.config.exception.GlobalExceptionHandler#handleException(Exception)`
- **实现意图**：兜底异常处理，所有未被具体捕获的 Exception 统一返回"未知错误"。如果是 `HttpMessageNotReadableException`（JSON 解析失败），返回参数格式错误提示。
- **返回参数**：

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 错误码（CommonCode.unknown；`HttpMessageNotReadableException` 时为 CommonCode.paraInvalid） |
| msg | String | 错误信息（「发生未知错误，请联系管理员!」；参数格式错误时「请检查参数，存在参数格式错误」） |
| data | JSONObject | 空对象 `{}` |
- **关键代码摘录**：
```java
@ExceptionHandler(Exception.class)
public ResponseResult<Object> handleException(Exception exception) {
    Logit.errorLog("请求处理未知错误:" + exception.getMessage(), exception);
    ResponseResult<Object> result = new ResponseResult<>(CommonCode.unknown.getValue(), "发生未知错误，请联系管理员!");
    result.setData(new HashMap<>());
    if (exception instanceof HttpMessageNotReadableException)
        result = new ResponseResult<>(CommonCode.paraInvalid.getValue(), "请检查参数，存在参数格式错误");
    return result;
}
```

---

### handleThrowable (`@ExceptionHandler(Throwable.class)`)

- **入口**：`cn.testin.config.exception.GlobalExceptionHandler#handleException(Throwable)`
- **实现意图**：终极兜底，捕获 Throwable（Error 级别异常），防止 tomcat 返回裸堆栈。
- **返回参数**：

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 错误码（CommonCode.unknown） |
| msg | String | 错误信息（「发生未知错误，请联系管理员!」） |
| data | JSONObject | 空对象 `{}` |
