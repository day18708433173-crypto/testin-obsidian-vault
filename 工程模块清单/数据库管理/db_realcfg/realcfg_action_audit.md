---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_action_audit

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

action 审计配置表：按模块配置需要审计的 action。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_action_audit`  (
                                         `id` int(11) NOT NULL AUTO_INCREMENT,
                                         `module_id` int(11) NOT NULL,
                                         `api_action` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                         `api_op` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                         `name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                         `descr` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                         `req_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                         `resp_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                         `status` int(11) NOT NULL,
                                         `createtime` bigint(20) NOT NULL,
                                         `updatetime` bigint(20) NOT NULL,
                                         PRIMARY KEY (`id`) USING BTREE,
                                         INDEX `module_id`(`module_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgActionAuditDAOImpl`（JDBC）← `ApiServiceImpl.listByActionAudit` ← 接口 [ApiActionCfg](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/ApiActionCfg.md)
