---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_audit_config

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

审计全局配置表。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_audit_config`  (
                                         `audit_key` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                         `audit_value` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                         `status` int(11) NULL DEFAULT NULL,
                                         `createtime` bigint(20) NULL DEFAULT NULL,
                                         `updatetime` bigint(20) NULL DEFAULT NULL,
                                         PRIMARY KEY (`audit_key`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

## 索引

- `PRIMARY KEY (`audit_key`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgAuditConfigDAOImpl`（JDBC）← `ApiServiceImpl.listByAuditConfig` ← 接口 [Audit](../../平台配置（real-cfg）/07-开放接口文档/用户与权限/Audit.md)
