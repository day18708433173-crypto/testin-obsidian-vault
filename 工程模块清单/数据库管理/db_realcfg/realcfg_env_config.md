---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_env_config

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

环境配置表：测试环境定义（hosts、数据源配置 db_config 等）。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_env_config`  (
                                       `id` int(11) NOT NULL AUTO_INCREMENT,
                                       `name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                       `eid` int(11) NOT NULL,
                                       `hosts` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '环境的host信息',
                                       `db_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
                                       `status` int(11) NOT NULL COMMENT '0：逻辑删除 1：正常状态 2：禁用状态',
                                       `createtime` bigint(20) NOT NULL,
                                       `updatetime` bigint(20) NOT NULL,
                                       `descr` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                       PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgEnvConfigDAOImpl`（JDBC）← `RealCfgEnvConfigServiceImpl` ← 接口 [EnvCfg](../../平台配置（real-cfg）/07-开放接口文档/项目与平台配置/EnvCfg.md)
- 被视图 [view_env_project](view_env_project.md) 引用
