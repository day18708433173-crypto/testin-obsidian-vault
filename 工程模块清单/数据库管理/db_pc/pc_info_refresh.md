---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# pc_info_refresh

## 用途

PC信息刷新辅助表。

## 所属数据库

db_pc

## DDL

```sql
CREATE TABLE `pc_info_refresh`  (
                                    `pool_id` int(11) NOT NULL,
                                    `ucomid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                    `vhost` int(11) NOT NULL,
                                    `os_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                    `os_version` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                    `action` int(11) NOT NULL,
                                    `debug_owner` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                    `browsers` varchar(2000) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                    `status` int(11) NOT NULL,
                                    `refreshtime` bigint(20) NOT NULL,
                                    `ip` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                    `location` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                    `source_rule` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                    `protocol` varchar(20) NULL COMMENT '上位机连接方式',
                                    `marks` varchar(2000) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                    `licences` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                    PRIMARY KEY (`pool_id`, `ucomid`) USING HASH
) ENGINE = MEMORY CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Fixed;
```

> DDL 来源：pocinit/src/mysql/db_pc.sql（命中）

## 设备控制中心 中的使用

PC信息刷新辅助表。
