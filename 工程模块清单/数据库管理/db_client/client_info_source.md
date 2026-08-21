---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# client_info_source

## 用途

客户端来源/资源标记表。ClientInfoSourceDAOImpl 操作。标记客户端的 source 类型。

## 所属数据库

db_client

## DDL

```sql
CREATE TABLE `client_info_source`  (
                                       `ucomid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
                                       `source` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
                                       `createtime` bigint(20) NULL DEFAULT NULL,
                                       `updatetime` bigint(20) NULL DEFAULT NULL,
                                       PRIMARY KEY (`ucomid`, `source`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_client.sql（命中）

## 设备控制中心 中的使用

客户端来源/资源标记表。ClientInfoSourceDAOImpl 操作。标记客户端的 source 类型。
