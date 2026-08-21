# service-SqlCtrl — SQL 语句管理（ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/source/SqlCtrl.java`
> 类级路由：`/source/SqlCtrl`（完整前缀 `/openapi/source/SqlCtrl`）
> 基类：`GenericBaseService`
> 业务：SQL 表达式 CRUD 管理（保存/分页查询/全量查询/删除）。

## 方法列表总表

| # | 路径 | 方法名 | 说明 |
|---|------|--------|------|
| 1 | `/save` | save | 保存或更新 SQL（id=null 新增，有 id 编辑） |
| 2 | `/selectPage` | selectPage | 分页查询（入参 `SqlDTO`） |
| 3 | `/selectAll` | selectAll | 查询所有（支持条件过滤，不入参分页信息） |
| 4 | `/delete` | delete | 按 ID 删除 SQL |

涉及表：`source_sql`。

---

## 方法详解

### 1. POST /save — 保存 SQL

**入参**（`Sql` 实体）：

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| sourceConfigId | Long | 是 | 所属数据源节点 ID |
| id | Long | 否 | 新增时传 null，编辑时传已有 ID |
| name | String | 否 | SQL 标题 |
| content | String | 否 | SQL 表达式 |
| dbName | String | 否 | 数据库实例名字 |
| envId / dbAlias / dbConfigId | Integer/String/Integer | 否 | 环境/库配置 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否保存成功 |

### 2. POST /selectPage — 分页查询

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id / eid / projectid / sourceConfigId / name | — | 否 | 过滤条件（SqlDTO） |
| current / size | Integer | 否 | 分页（Query 基类，默认 1/10） |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<SqlVO\> | SQL 视图列表 |
| data.list[].id | Long | SQL id |
| data.list[].sourceConfigId | Long | 所属数据源 id |
| data.list[].name | String | SQL 名字 |
| data.list[].content | String | SQL 内容 |
| data.list[].dbName | String | 数据库实例名字 |
| data.list[].envId | Integer | 环境 id |
| data.list[].dbAlias | String | 数据库别名 |
| data.list[].dbConfigId | Integer | 数据配置 id |
| data.list[].status | Integer | 状态 1 正常 0 删除 |
| data.list[].createtime / updatetime | Long | 创建/更新时间 |
| data.page | Integer | 页码 |
| data.pageSize | Integer | 每页条数 |
| data.totalRow | Long | 总记录数 |
| data.totalPage | Long | 总页数 |

返回 `IPage<SqlVO>`。

### 3. POST /selectAll — 全量查询

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| （SqlDTO 字段） | — | 否 | 过滤条件 |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.list | List\<SqlVO\> | SQL 视图列表（每项字段同 selectPage：id/sourceConfigId/name/content/dbName/envId/dbAlias/dbConfigId/status/createtime/updatetime） |

返回 `List<SqlVO>`（不带分页），支持条件过滤。

### 4. POST /delete — 删除

**请求参数**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Long | 是 | SQL id |

**返回参数**

| 字段 | 类型 | 说明 |
|---|---|---|
| code | Integer | 0 成功，非 0 失败 |
| msg | String | 提示信息 |
| data.data | Boolean | 是否删除成功 |

按 `id` 逻辑删除 SQL 记录（status→0）。
