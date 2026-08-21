---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: ApiServlet
---

# Sql（db 包）

## 职责
按已保存的数据库配置（configId）执行 SQL：先做企业/项目组权限校验，再区分关系型（进程内执行）与 MongoDB/Redis（XML-RPC 转发）。

- 源码：`real-controlcenter/src/main/java/cn/testin/service/db/Sql.java`
- 基类：`GenericBaseService`（注入 dbconfigapi、dbRsaApi）
- 对比：[NewSql](长连接/NewSql.md) 为新版（按 envId+dbAlias 定位配置），本类为旧版（按 configId）。

## op 一览表

| op | 说明 |
|---|---|
| execute | 执行 SQL/命令（含权限校验） |

---

### execute (`Sql.execute`)
- **入口**：ApiServlet，action/op（action=db，op=Sql.execute）
- **实现意图**：以 configId 加载库配置，校验企业（eid）与项目组（projectid）使用权后执行语句；SELECT 返回结果集，其他语句返回影响行数。
- **请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | int | 是 | 企业 ID（须与配置一致） |
| projectid | int | 是 | 项目组 ID（配置 projectId=0 表示全组可用） |
| configId | int | 是 | 数据库配置 ID |
| sql | String | 是 | SQL/命令文本 |
| timeout | int | 否 | 超时，缺省取配置值 |
| dbSecretPwd | String | 否 | 传此键时使用密文密码并解密 |

- **返回参数**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| code | Integer | 状态码，0 成功 |
| msg | String | 提示信息 |
| data | Object | 返回数据对象 |
| data.list | JSONArray | 结果集：SELECT 为行记录数组；增删改为 `[{"affectedRows": 影响行数}]`；Mongo/Redis 为 `[{"result": ...}]` |

- NoSQL 分支顶层额外回显 action/op 字段。
- **处理流程**：
```mermaid
flowchart TD
    A[校验 eid/projectid/sql] --> B[dbconfigapi.getDbConfig configId]
    B -- 空 --> C[GeneralException paraInvalid]
    B --> D{eid 匹配? projectid 匹配?} -- 否 --> E[dbConfigFailed]
    D -- 是 --> F{Mongo/Redis/RedisCluster?}
    F -- 是 --> G[redis.execute](XmlRpcService.mongodb/redis.execute)
    F -- 否 --> H[IDbSqlServiceImpl.<type>]
    H --> I{isSelectSql?} -- 是 --> J[query 返回行集]
    I -- 否 --> K[update 返回 affectedRows]
```
- **权限校验**（新版 NewSql 无此步骤）：
  - eid/projectid 缺失、sql 空 → 抛 `GeneralException(paraInvalid)`；配置为空 → 抛 `GeneralException(paraInvalid)`；
  - eid 匹配：`!eid.equals(dbConfig.getEid())` → 抛 `GeneralException(dbConfigFailed)`；
  - projectid 匹配：`dbConfig.getProjectId() != 0 && !projectid.equals(dbConfig.getProjectId())` → 返回 paraInvalid（"项目组无权使用"）；**projectId==0 表示所有项目组都可用**；
  - 注：源码里 eid 校验写了两遍（一次抛 dbConfigFailed 异常、一次 return paraInvalid），后一处是冗余死代码。
- **按数据库类型分流**（`DbTypeEnum.valOf(dbConfig.getTypeId())`，未知 typeId → 抛 `dbNotSupport`）：
  - **NoSQL 分支（REDISDB=6 / MONGODB=7 / REDISCLUSTERDB=9）→ XML-RPC 转发，非本进程执行**：
    - 组装 `dbConfigJson`（dbAddress/dbPort/dbName/dbUser/timeout/dbPwd，timeout 缺省取 `dbConfig.getTimeout()`）；
    - 密码处理：请求带非空 `dbSecretPwd` → `dbRsaApi.decryptPassword(dbConfig.getDbSecretPwd())` 解密密文；否则用明文 `dbConfig.getDbPassWord()`；
    - 按类型调 `MongoDBApi.execute` / `RedisDBApi.execute` / `RedisClusterDBApi.execute`，`{dbConfig, sql}` 经 XML-RPC 转发到 xmlrpc-service；
    - 返回结构规整：远端 `data` 三种形态（list/object/result）统一包成 `{result: <原值>}`，最终 `data.list=[{result:...},...]`，顶层回显 action/op。
  - **关系型分支（其余 11 种）→ 进程内执行**：
    - Bean 名 = `IDbSqlServiceImpl.` + `dbTypeEnum.getName().toLowerCase()`，`SpringHelper.getBean` 取对应库实现，取不到 → 抛 `dbNotSupport`；
    - 构建 `DBConnectionDTO`（dbAddress/dbPort/dbName/dbUser/dbPwd/timeout）；
    - `isSelectSql(sql)` 分流：SELECT → `query` 行集 + `listToResList` 规整；增删改 → `update`，返回 `data.list[0].affectedRows`；
    - 关系型实现注册于 `Spring-Service.xml`：mysql/oracle/postgresql/vastbase/db2/sqlserver/as400/oceanbase/dm/opengauss；**vastbase 复用 PostgreSQL 实现，sybase 未注册 → dbNotSupport**。
- **调用链**：[平台配置](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md)（dbconfigapi.getDbConfig 读库配置）；xmlrpc-service（XML-RPC `db` 服务 mongodb.execute/redis.execute/rediscluster.execute）。
- **涉及表与 SQL**：配置表由 平台配置 维护（RealCfgDbConfig）；目标库执行调用方传入的任意 SQL。
- **异常与校验**：eid/projectid 缺失、sql 空 → GeneralException(paraInvalid)；配置不存在 → GeneralException(paraInvalid)；企业/项目组无权 → dbConfigFailed（eid 不匹配抛异常）或 paraInvalid（projectid 不匹配 return）；typeId 未知或关系型 Bean 缺失（如 sybase）→ dbNotSupport；执行异常 GeneralException 原样上抛。
- **关键代码摘录**：
```java
// real-controlcenter/src/main/java/cn/testin/service/db/Sql.java
RealCfgDbConfig dbConfig = dbconfigapi.getDbConfig(configId, null, null);
if (dbConfig.getProjectId() != 0 && !projectid.equals(dbConfig.getProjectId())) {
    return ApiUtil.getJSONobj(apirequest, CommonCode.paraInvalid.getValue(), msg).toString();
}
if (idbsqlservice.isSelectSql(sql)) {
    result = idbsqlservice.query(dbConnectionDTO, sql);
} else {
    rowNum = idbsqlservice.update(dbConnectionDTO, sql);
}
```

---

## 依赖汇总
- 外部服务：[平台配置](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md)（库配置）、xmlrpc-service（NoSQL 执行）
- 主要表：目标业务库任意表（由调用方 SQL 决定）
