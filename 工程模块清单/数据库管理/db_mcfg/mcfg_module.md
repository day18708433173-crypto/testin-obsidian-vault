---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# mcfg_module

- 数据库：`db_mcfg`
- 对象类型：表

## 用途

openapi 模块表：按 mkey 标识业务模块。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `mcfg_module`  (
                                `id` int(11) NOT NULL,
                                `mkey` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                `name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                `http_hosts` varchar(2048) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `rpc_prefix_name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `distributed` int(11) NOT NULL COMMENT '是否是数据分布部署',
                                `rule_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '数据分布部署的规则',
                                `descr` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                `status` int(11) NOT NULL,
                                `createtime` bigint(20) NOT NULL,
                                `updatetime` bigint(20) NOT NULL,
                                PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `McfgModuleDAOImpl`（JDBC）← `ModuleServiceImpl` ← 接口 [ModuleCfg](../../平台配置（real-cfg）/07-开放接口文档/用户与权限/ModuleCfg.md)（get/list）
