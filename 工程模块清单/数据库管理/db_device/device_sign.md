---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_sign

## 用途

设备签名/标记表。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_sign`  (
                                `id` bigint(20) NOT NULL AUTO_INCREMENT,
                                `deviceid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                `ucomid` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `base_imei` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `base_androidid` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `base_mac` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `base_serial_number` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `status` int(11) NOT NULL,
                                `createtime` bigint(20) NOT NULL,
                                PRIMARY KEY (`id`) USING BTREE,
                                UNIQUE INDEX `base_imei`(`base_imei`, `base_androidid`, `base_mac`) USING BTREE,
                                INDEX `base_imei_2`(`base_imei`, `base_mac`) USING BTREE,
                                INDEX `base_androidid`(`base_androidid`, `base_mac`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备签名/标记表。
