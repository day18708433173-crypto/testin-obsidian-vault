---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_info_extend

## 用途

设备扩展信息表（MCU版本、机器版本、ICC ID、ROM刷写状态等）。ExtendDeviceWorkerThread 维护其内存缓存过期清理。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_info_extend`  (
                                       `ext_deviceid` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                       `ucomid` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                       `ip` varchar(15) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                       `port` int(11) NOT NULL,
                                       `location` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                       `descr` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                       `status` int(11) NOT NULL,
                                       `createtime` bigint(20) NOT NULL,
                                       `updatetime` bigint(20) NOT NULL,
                                       PRIMARY KEY (`ext_deviceid`) USING BTREE,
                                       UNIQUE INDEX `ip_port`(`ip`, `port`) USING BTREE,
                                       INDEX `ucomid`(`ucomid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备扩展信息表（MCU版本、机器版本、ICC ID、ROM刷写状态等）。ExtendDeviceWorkerThread 维护其内存缓存过期清理。
