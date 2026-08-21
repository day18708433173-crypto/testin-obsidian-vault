---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_db_rsa

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

数据源 RSA 密钥配置表。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_db_rsa`  (
                                   `id` int(11) NOT NULL AUTO_INCREMENT,
                                   `pub` varchar(1000) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '公钥',
                                   `pri` varchar(1000) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '私钥',
                                   `status` tinyint(4) NOT NULL DEFAULT 1,
                                   `createtime` bigint(20) NOT NULL,
                                   `updatetime` bigint(20) NULL DEFAULT NULL,
                                   PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgDbRsaImpl`（JDBC）← `RealCfgDbRsaServiceImpl` ← 接口 [DbRsa](../../平台配置（real-cfg）/07-开放接口文档/数据源与代码配置/DbRsa.md)
