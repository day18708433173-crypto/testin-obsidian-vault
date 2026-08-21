---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: ApiServlet
---

# NewSql（db 包）

## 职责
新版 SQL 执行接口：按环境（envId）+ 库别名（dbAlias）定位数据库配置并执行语句。与 [Sql](长连接/Sql.md) 的区别是不再做企业/项目组归属校验，面向环境化配置的新模型。

- 源码：`real-controlcenter/src/main/java/cn/testin/service/db/NewSql.java`
- 基类：`GenericBaseService`（注入 dbconfigapi、dbRsaApi）

## op 一览表

| op | 说明 |
|---|---|
| execute | 执行 SQL/命令（按 envId+dbAlias） |

---

### execute (`NewSql.execute`)
- **入口**：ApiServlet，action/op（action=newSql，op=NewSql.execute）
- **实现意图**：加载 envId+dbAlias 对应的库配置，关系型库进程内执行（SELECT/增删改分流），MongoDB/Redis 经 XML-RPC 转发，返回结构统一规整为 `data.list[{result}]`。
- **请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| envId | int | 是 | 环境 ID（>0） |
| dbAlias | String | 是 | 库别名 |
| sql | String | 是 | SQL/命令文本 |
| timeout | int | 否 | 超时，缺省取配置值 |
| dbSecretPwd | String | 否 | 传此键时改用密文密码解密 |

- **返回参数**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 返回数据对象 |
| data.list | JSONArray | 结果集：SELECT 为行记录数组；增删改为 `[{"affectedRows": n}]`；Mongo/Redis 为 `[{"result": ...}]` |

- NoSQL 分支顶层额外回显 action/op 字段。
- **处理流程**：
```mermaid
flowchart TD
    A[校验 sql/dbAlias/envId] --> B[dbconfigapi.getDbConfig null,dbAlias,envId]
    B -- 空 --> C[GeneralException]
    B --> D{DbTypeEnum 校验} -- 不支持 --> E[dbNotSupport]
    D --> F{Mongo/Redis/RedisCluster?}
    F -- 是 --> G[XmlRpcService.execute](XmlRpcService.execute)
    F -- 否 --> H[IDbSqlServiceImpl.<type>] --> I{isSelectSql?}
    I -- 是 --> J[query 行集] 
    I -- 否 --> K[update affectedRows]
```
- **按数据库类型分流**（`DbTypeEnum.valOf(dbConfig.getTypeId())` 先判定类型，未知 typeId → 抛 `dbNotSupport`）：
  - **NoSQL 分支（REDISDB=6 / MONGODB=7 / REDISCLUSTERDB=9）→ XML-RPC 转发，非本进程执行**：
    - 组装 `dbConfigJson`：`dbAddress/dbPort/dbName/dbUser/timeout/dbPwd`（timeout 缺省取 `dbConfig.getTimeout()`）；
    - 密码处理：请求带非空 `dbSecretPwd` → `dbRsaApi.decryptPassword(dbConfig.getDbSecretPwd())` 解密密文；否则用明文 `dbConfig.getDbPassWord()`；
    - 按类型调 `MongoDBApi.execute` / `RedisDBApi.execute` / `RedisClusterDBApi.execute`，把 `{dbConfig, sql}` 经 XML-RPC 转发到 xmlrpc-service；
    - 返回结构规整：远端 `data` 可能是 `list`（数组）/`object`（对象）/`result`（单值）三种形态，统一逐项包成 `{result: <原值>}`，最终 `data.list=[{result:...},...]`，顶层回显 action/op。
  - **关系型分支（其余 11 种）→ 进程内执行**：
    - Bean 名 = `IDbSqlServiceImpl.` + `dbTypeEnum.getName().toLowerCase()`，`SpringHelper.getBean` 动态取对应库实现，取不到 → 抛 `dbNotSupport`；
    - 构建 `DBConnectionDTO`（dbAddress/dbPort/dbName/dbUser/dbPwd/timeout，timeout 缺省取配置值）；
    - `isSelectSql(sql)` 分流：SELECT → `query` 行集 + `listToResList` 规整成 `data.list`；增删改 → `update`，返回 `data.list[0].affectedRows`。
    - 关系型实现由 `Spring-Service.xml` 注册：mysql/oracle/postgresql/vastbase/db2/sqlserver/as400/oceanbase/dm/opengauss 各一个 bean；**vastbase 复用 PostgreSQL 实现，sybase 未注册 → 会 dbNotSupport**。
- **调用链**：[平台配置](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md)（dbconfigapi.getDbConfig）；xmlrpc-service（mongodb.execute / redis.execute / rediscluster.execute）。
- **涉及表与 SQL**：配置表由 平台配置 维护；目标库任意 SQL。
- **异常与校验**：reqjson 为空、sql/dbAlias 空、envId ≤ 0 → paraInvalid（注意 envId 的报错文案误写成 "configId is invalid!"，属复制粘贴遗留）；配置为空 → GeneralException(paraInvalid)；typeId 未知或关系型 Bean 缺失（如 sybase）→ dbNotSupport；执行异常 GeneralException 原样上抛。
- **关键代码摘录**：
```java
// real-controlcenter/src/main/java/cn/testin/service/db/NewSql.java
RealCfgDbConfig dbConfig = dbconfigapi.getDbConfig(null, dbAlias, envId);
...
StringBuffer dbBeanName = new StringBuffer();
dbBeanName.append("IDbSqlServiceImpl").append(".").append(dbTypeEnum.getName().toLowerCase());
IDbSqlService idbsqlservice = (IDbSqlService) SpringHelper.getBean(dbBeanName.toString());
```

---

## 依赖汇总
- 外部服务：[平台配置](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md)（库配置）、xmlrpc-service（NoSQL 执行）
- 主要表：目标业务库任意表（由调用方 SQL 决定）
