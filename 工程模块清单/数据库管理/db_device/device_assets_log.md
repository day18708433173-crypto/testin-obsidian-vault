---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_assets_log

## 用途

设备资产操作日志表，记录资产变更历史。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_assets_log`  (
                                      `id` bigint(20) NOT NULL AUTO_INCREMENT,
                                      `deviceid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                      `assets_num` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '资产编号',
                                      `region` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                      `owner` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                      `descr` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
                                      `operator` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                      `additional_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
                                      `status` int(11) NOT NULL,
                                      `createtime` bigint(20) NOT NULL,
                                      `mark1` varchar(1024) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '备用字段1',
                                      `mark2` varchar(1024) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '备用字段2',
                                      PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备资产操作日志表，记录资产变更历史。
