---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# pc_info_source

## 用途

PC来源/资源标记表。PcInfoSourceDAOImpl 操作。标记上位机的 source 类型。

## 所属数据库

db_pc

## DDL

```sql
CREATE TABLE `pc_info_source`  (
                                   `ucomid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
                                   `source` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
                                   `createtime` bigint(20) NULL DEFAULT NULL,
                                   `updatetime` bigint(20) NULL DEFAULT NULL,
                                   PRIMARY KEY (`ucomid`, `source`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_pc.sql（命中）

## 设备控制中心 中的使用

PC来源/资源标记表。PcInfoSourceDAOImpl 操作。标记上位机的 source 类型。
