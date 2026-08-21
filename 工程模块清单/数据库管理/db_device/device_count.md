---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_count

## 用途

设备数量趋势统计表。DeviceCountThread 每小时写入各项目设备数量，按 createtime 整点存储。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_count`  (
                                 `id` int(11) NOT NULL AUTO_INCREMENT,
                                 `eid` int(11) NULL DEFAULT NULL,
                                 `projectid` int(11) NULL DEFAULT NULL,
                                 `device_count` int(11) NULL DEFAULT NULL,
                                 `createtime` bigint(20) NULL DEFAULT NULL,
                                 PRIMARY KEY (`id`) USING BTREE,
                                 INDEX `eid_projectid`(`eid`, `projectid`) USING BTREE,
                                 INDEX `createtime_eid_project`(`createtime`, `eid`, `projectid`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备数量趋势统计表。DeviceCountThread 每小时写入各项目设备数量，按 createtime 整点存储。
