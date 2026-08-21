---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: 接口文档
entry: 长连接协议
---

# Sql

> mkey: `script` | 包路径: `cn.testin.trans.controller.req.script.Sql`

SQL 执行命令（旧版），支持关系型数据库和 MongoDB/Redis。带有企业/项目组权限校验。

## execute

### 协议命令

```
{ "mkey": "script", "op": "Sql.execute", "reqid": "xxx", "data": { "configId": 1, "projectId": 10, "sql": "SELECT * FROM users", "timeout": 5000 } }
```

### 实现意图

通过 configId 定位数据库配置，执行 SQL 语句。额外做了企业归属（eid）和项目组（projectId）的权限校验。

### 请求消息字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| data.configId | int | 是 | 数据库配置 ID |
| data.projectId | int | 是 | 项目组 ID（用于权限校验） |
| data.sql | String | 是 | SQL 语句 |
| data.timeout | int | 否 | 超时时间 |

### 权限校验

1. 通过 `projectapi.get(projectId)` 获取项目组信息
2. 校验 `dbConfig.eid == projectInfo.eid`（同企业）
3. 校验 `dbConfig.projectId == 0 || projectId == dbConfig.projectId`（同项目组或公开配置）

### 调用链

```
trans.controller.req.script.Sql.execute(Session, RequestContext)
  → dbconfigapi.getDbConfig(configId, null, null)     // [real-cfg](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md)
  → projectapi.get(projectId)                          // [user-manager](../../../平台基础功能服务/00-首页.md)
  // 关系型 DB
  → IDbSqlService.query/update(DBConnectionDTO, sql)
  // NoSQL
  → MongoDBApi.execute(dataJson)
  → RedisDBApi.execute(dataJson)
```

### 涉及表/SQL

- `realcfg_db_config` — 数据库配置
- [user-manager](../../../平台基础功能服务/00-首页.md) — 项目组查询

### 异常处理

- configId 无效 → `paraInvalid`
- projectId 无效 → `paraInvalid`
- 企业无权限 → `dbConfigFailed`，"The enterprise has no right to use!"
- 项目组无权限 → `dbConfigFailed`，"The project has no right to use!"
- dbType 不受支持 → `dbNotSupport`
