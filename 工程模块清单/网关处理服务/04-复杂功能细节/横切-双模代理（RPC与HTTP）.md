# 双模代理（RPC与HTTP）

> 网关在 `executeRequest` 阶段根据 `McfgModule.httpHosts` 是否为空选择 RPC（ICE）或 HTTP 转发。V3 还多一个 `passThroughType=1` 透传变体。本文拆解三种转发模式的判定、代码路径、配置来源和差异。

## 转发模式判定

```
executeRequest(McfgModule, reqJson, HttpServletRequest)
  │
  ├── httpHosts 为空？
  │     YES → RPC 模式
  │     │      RpcServiceImpl.doPress(rpcPrefixName, reqJson)
  │     │      └→ cn.testin.common.service.util.ApiUtil.doPress(prefixName, json, true)
  │     │         └→ ICE Locator 路由到 RealCfg / UserManager 等 V1 服务
  │     │
  │     NO  → HTTP 模式（POST）
  │     │      HttpServiceImpl.doPress(request, httpHost, reqJson)
  │     │      └→ HttpURLConnection POST → 目标服务
  │     │
  │     (V3 特殊路径) passThroughType == 1？
  │            YES → HTTP 透传模式
  │                   HttpServiceImpl.doPressWithPassThrough(request, httpHost)
  │                   └→ 原始 HTTP 方法/headers/query/body 全部保留
```

## RPC 模式（ICE）

### 配置来源

- **rpcPrefixName**：来自 `McfgModule.rpcPrefixName`（如 `RealCfg`、`UserManager`、`device_monitor`、`script`、`RealTest`、`Plan`、`common`）
- **ICE Locator**：`src/main/resources/iceclient.properties`（零 C Ice 网格配置）
- **RPC 调用链**：`RpcServiceImpl.doPress` → `cn.testin.common.service.util.ApiUtil.doPress(prefixName, reqJson, true)` → ICE 动态代理

### 代码路径

`business/impl/RpcServiceImpl.java`

```java
public ApiJsonResponse doPress(String prefixName, String reqJson) throws GeneralException {
    return ApiUtil.doPress(prefixName, reqJson, true);
}
```

`ApiUtil.doPress` 是 `real-common` 外部依赖的方法，内部通过 ICE Locator 解析 `prefixName` 对应的服务端点和接口方法，将 `reqJson` 作为参数传入。`true` 参数表示使用 JSON 序列化。

### 适用场景

- **RealCfg**：McfgApp/McfgModule/McfgApi/McfgRole 配置查询
- **UserManager**：用户/项目/企业/在线信息查询
- **其他 V1 ICE 服务**：如 device_monitor、Plan、common

### 特点

- 无连接池，每次调用新建 ICE 代理
- 依赖 ICE Locator 的网格寻址，目标服务地址不在网关配置中
- 同步阻塞调用

## HTTP 模式（POST）

### 配置来源

- **httpHosts**：来自 `McfgModule.httpHosts`（`;` 分隔多地址，随机取一实现负载均衡）
- **代理 headers**：`application.properties` 中的 `httpservice.proxy.headers`（以 `[` 开头的 JSON 数组）
- **超时**：环境变量 `OPENAPI_CONNECT_TIMEOUT` / `OPENAPI_READ_TIMEOUT`（默认 1,200,000ms）

### 代码路径

`business/impl/HttpServiceImpl.java`

```java
public ApiJsonResponse doPress(HttpServletRequest request, String targetUrl, String reqJson) {
    HttpURLConnection conn = (HttpURLConnection) new URL(targetUrl).openConnection();
    conn.setConnectTimeout(connectTimeout);    // 默认 1,200,000ms
    conn.setReadTimeout(readTimeout);          // 默认 1,200,000ms
    conn.setRequestMethod("POST");
    conn.setDoOutput(true);
    conn.setRequestProperty("Content-Type", "application/json; charset=UTF-8");

    // 设置代理 headers
    HttpHeaderUtil.setProxyHeaders(conn, request, proxyHeaders);

    // 写 body
    OutputStream os = conn.getOutputStream();
    os.write(reqJson.getBytes("UTF-8"));

    // 读响应（支持 GZIP）
    InputStream is = conn.getInputStream();
    if ("gzip".equals(conn.getContentEncoding())) {
        is = new GZIPInputStream(is);
    }
    String result = IOUtils.toString(is, "UTF-8");
    return new ApiJsonResponse(result);
}
```

### 特点

- 每次 new `HttpURLConnection`，未复用连接
- 支持 GZIP 自动解压
- 通过 header 转发实现上下文传递（`HttpHeaderUtil.setProxyHeaders`）
- 仅支持 POST 方法（V1/V2 的 HTTP 转发）

## HTTP 透传模式（V3 passThroughType=1）

### 判定条件

1. V3 请求（`ApiV3ProxyServlet`）
2. `McfgApi.protocolConfig.passThroughType == 1`
3. `McfgModule.httpHosts` 非空

### 代码路径

`business/impl/HttpServiceImpl.java`

```java
public String doPressWithPassThrough(HttpServletRequest request, String targetHost) {
    // 1. 保留原始 HTTP 方法
    String method = request.getMethod();
    HttpURLConnection conn = (HttpURLConnection) new URL(targetUrl).openConnection();
    conn.setRequestMethod(method);

    // 2. 合并 query 参数
    if (request.getQueryString() != null) {
        targetUrl += "?" + request.getQueryString();
    }

    // 3. 转发 headers
    Enumeration<String> headerNames = request.getHeaderNames();
    while (headerNames.hasMoreElements()) {
        String name = headerNames.nextElement();
        if (isAllowedHeader(name)) {
            conn.setRequestProperty(name, request.getHeader(name));
        }
    }

    // 4. 流式转发 body（支持 multipart/form-data）
    if (request.getContentType() != null
        && request.getContentType().contains("multipart/form-data")) {
        conn.setRequestProperty("Content-Type", request.getContentType());
        InputStream is = request.getInputStream();
        OutputStream os = conn.getOutputStream();
        IOUtils.copy(is, os);
    } else if (body != null) {
        // 写普通 body
        conn.getOutputStream().write(body.getBytes("UTF-8"));
    }

    // 5. 回传响应（含 Set-Cookie 和 octet-stream）
    String setCookie = conn.getHeaderField("Set-Cookie");
    String contentType = conn.getContentType();
    if ("application/octet-stream".equals(contentType)) {
        // 文件下载：设置 Content-Disposition + 写入原始字节
        response.setContentType("application/octet-stream");
        response.setHeader("Content-Disposition",
            conn.getHeaderField("Content-Disposition"));
        IOUtils.copy(conn.getInputStream(), response.getOutputStream());
        return null;  // 已直接写入 response
    }
    return IOUtils.toString(conn.getInputStream(), "UTF-8");
}
```

### 与 POST HTTP 模式的关键差异

| 维度 | HTTP POST | HTTP 透传 |
|---|---|---|
| HTTP 方法 | 固定 POST | 保留原始（GET/POST/PUT/DELETE） |
| query 参数 | 不处理 | 合并到目标 URL |
| 请求 headers | 仅转发代理 headers | 转发所有允许的 headers |
| body 类型 | JSON only | JSON / multipart/form-data |
| 响应 Set-Cookie | 忽略 | 回传 |
| 文件下载 | 不支持 | 支持（octet-stream + Content-Disposition） |
| 适用接口 | 全部 V1/V2 HTTP 接口 | V3 passThroughType=1 接口 |
| 编排处理 | 前后都做编排 | 完全跳过编排 |

### 适用场景

- **real_task 全部接口**（`/*` passThroughType=1，纯 V3 透传）
- **script `/scripts/*`**（passThroughType=1）
- **其他 V3 原生 REST 服务**
