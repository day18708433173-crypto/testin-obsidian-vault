---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_log_info

## 用途

设备日志信息表，存储设备运行日志摘要。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_log_info`  (
                                    `id` bigint(20) NOT NULL AUTO_INCREMENT,
                                    `ucomid` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                    `deviceid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                    `subtaskid` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                    `sub_subtaskid` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                    `model_id` int(11) NULL DEFAULT NULL,
                                    `device_model_id` int(11) NULL DEFAULT NULL,
                                    `release_ver` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                    `code` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                    `msg` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                    `descr` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
                                    `status` int(11) NOT NULL,
                                    `createtime` bigint(20) NOT NULL,
                                    PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备日志信息表，存储设备运行日志摘要。
