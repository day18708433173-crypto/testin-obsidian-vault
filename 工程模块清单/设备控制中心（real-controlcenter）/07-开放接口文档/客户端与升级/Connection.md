---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: ApiServlet
---

# Connection（db 包）

## 职责
数据库连接连通性测试。关系型库走本进程 `IDbSqlService` 实现；MongoDB/Redis/Redis 集群转发给外部 XML-RPC 服务执行。

- 源码：`real-controlcenter/src/main/java/cn/testin/service/db/Connection.java`
- 基类：`GenericBaseService`（注入 dbRsaApi）

## op 一览表

| op | 说明 |
|---|---|
| testTry | 测试数据库连接 |

---

### testTry (`Connection.testTry`)
- **入口**：ApiServlet，action/op（action=db，op=Connection.testTry）
- **实现意图**：在不保存配置的情况下验证一组数据库连接参数是否可用；支持密文密码解密。
- **请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| dbTypeName | String | 是 | 数据库类型（DbTypeEnum，去空格） |
| dbAddress | String | 是 | 地址 |
| dbPort | int | 是 | 端口 |
| dbName | String | 否 | 库名 |
| dbUser | String | 否 | 用户名 |
| dbPwd | String | 否 | 明文密码（优先） |
| dbSecretPwd | String | 否 | 密文密码（无明文时 RSA 解密） |
| timeout | int | 否 | 超时，默认 CommonConstant.DEFAULT_TIMEOUT |

- **返回参数**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 返回数据对象 |
| data.result | Integer | 1 成功 / 0 失败 / 负数错误码 |
| data.msg | String | 结果描述（Mongo/Redis 分支或关系型异常时返回） |
- **处理流程**：
```mermaid
flowchart TD
    A[解析连接参数] --> B{dbPwd 明文?} -- 否 --> C[dbRsaApi.decryptPassword 解密 dbSecretPwd]
    A -- 是 --> D[DbTypeEnum.valOf 校验]
    C --> D
    D --> E{Mongo/Redis/RedisCluster?}
    E -- 是 --> F[redis.testTry](XmlRpcService.mongodb/redis.testTry)
    E -- 否 --> G[SpringHelper.getBean IDbSqlServiceImpl.<type>] --> H[idbsqlservice.testTry]
    F --> I[规整 code/msg 返回]
    H --> I
```
- **调用链**：xmlrpc-service（XML-RPC `db` 服务：`mongodb.testTry`/`redis.testTry`/`rediscluster.testTry`）；关系型库为进程内 `IDbSqlService` Bean。
- **涉及表与 SQL**：无（仅连接探测）。
- **异常与校验**：dbTypeName 无法识别 → GeneralException(paraInvalid)；Bean 不存在 → GeneralException(dbNotSupport)；关系型连接异常 → result=错误码、msg=异常信息。
- **关键代码摘录**：
```java
// real-controlcenter/src/main/java/cn/testin/service/db/Connection.java
if (!reqjson.isNull("dbSecretPwd") && !StringUtils.isBlank(reqjson.getString("dbSecretPwd"))) {
    dbPwd = dbRsaApi.decryptPassword(dbSecretPwd);
}
...
result = MongoDBApi.testTry(dataJson);   // XML-RPC: db / mongodb.testTry
```

---

## 依赖汇总
- 外部服务：xmlrpc-service（MongoDB/Redis 探测）
- 主要表：无
