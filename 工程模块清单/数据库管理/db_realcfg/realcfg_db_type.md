---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_db_type

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

数据源类型字典表（MySQL/Redis/MongoDB 等类型枚举落库）。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_db_type`  (
                                    `type_id` int(11) NOT NULL AUTO_INCREMENT,
                                    `type_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                    `status` tinyint(4) NOT NULL DEFAULT 1,
                                    `createtime` bigint(20) NOT NULL,
                                    `updatetime` bigint(20) NULL DEFAULT NULL,
                                    PRIMARY KEY (`type_id`) USING BTREE,
                                    UNIQUE INDEX `type_id_type_name`(`type_id`, `type_name`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`type_id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgDbTypeDAOImpl`（JDBC）← `RealCfgDbTypeServiceImpl` ← 接口 [DbCfg](../../平台配置（real-cfg）/07-开放接口文档/数据源与代码配置/DbCfg.md)
