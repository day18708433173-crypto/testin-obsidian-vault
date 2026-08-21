---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_env_enterprise

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

环境-企业/项目关联表：环境可被哪些企业项目使用。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_env_enterprise`  (
                                           `env_id` int(11) NOT NULL,
                                           `eid` int(11) NOT NULL,
                                           `project_id` int(11) NOT NULL,
                                           `createtime` bigint(20) NOT NULL,
                                           `updatetime` bigint(20) NOT NULL,
                                           `project_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- DDL 中未见显式索引

## 被哪些接口/mapper 方法使用

- `RealcfgEnvConfigDAOImpl`（JDBC）← `RealCfgEnvConfigServiceImpl` ← 接口 [EnvCfg](../../平台配置（real-cfg）/07-开放接口文档/项目与平台配置/EnvCfg.md)
- 被视图 [view_env_project](view_env_project.md) 引用
