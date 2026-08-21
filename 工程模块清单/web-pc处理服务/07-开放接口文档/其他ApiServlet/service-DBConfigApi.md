# service-DBConfigApi — 数据库环境配置查询

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/realcfg/DBConfigApi.java`
> 类型：远端代理（→ RealCfg 服务）
> 转发方式：V1 ApiServlet 前缀（`ApiUtil.doPress("RealCfg", reqJson)`，action/op 反射路由）

## 方法列表

### 1. list — 按环境查询数据库配置

```java
public List<RealCfgDbConfig> list(Integer eid, Integer envId, String dbAlias) throws GeneralException
```

**用途**：查询指定环境下某 dbAlias 的数据库连接配置（报告生成时用于 DB 断言/数据校验取连接信息）。

**转发目标**：`action=cfg, op=DbCfg.listByEnvId`（RealCfg）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 id |
| envId | Integer | 是 | 环境 id，null 抛 `paraInvalid` |
| dbAlias | String | 是 | 数据库别名，空抛 `paraInvalid` |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | RealCfgDbConfig | 数据库连接配置对象（字段见 RealCfg 服务 `DbCfg.listByEnvId`，代码未确认） |

**调用者**：
- `ReportServiceImpl.java`（`new DBConfigApi().list(eid, envId, dbAlias)`，报告步骤 DB 校验）

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealCfg 服务](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md)
- [service-EnvConfigApi](service-EnvConfigApi.md)
