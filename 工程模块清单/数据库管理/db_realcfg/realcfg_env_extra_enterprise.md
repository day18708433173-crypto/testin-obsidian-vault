---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_env_extra_enterprise

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

环境企业扩展配置表：按 env_id 保存环境的企业级附加配置。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE
    `db_realcfg`.`realcfg_env_extra_enterprise` (
                                                    `id` int (11) NOT NULL AUTO_INCREMENT,
                                                    `env_id` int (11) NOT NULL,
                                                    `project_id` int (11) NOT NULL,
                                                    `type` int (11) NOT NULL,
                                                    `content` varchar (512) NOT NULL,
                                                    `createtime` bigint (20) NOT NULL,
                                                    `updatetime` bigint (20) NOT NULL,
                                                    PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB DEFAULT CHARSET = utf8 ROW_FORMAT = COMPACT;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgEnvConfigDAOImpl`（JDBC，`RealCfgEnvConfigExtraRowMapper`，按 env_id 增删查）← `RealCfgEnvConfigServiceImpl` ← 接口 [EnvCfg](../../平台配置（real-cfg）/07-开放接口文档/项目与平台配置/EnvCfg.md)
