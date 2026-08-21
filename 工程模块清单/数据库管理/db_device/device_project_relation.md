---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_project_relation

## 用途

设备-项目关联表，记录设备与项目的归属关系。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_project_relation` (
                                           `id` bigint(11) NOT NULL AUTO_INCREMENT,
                                           `deviceid` varchar(128) DEFAULT NULL,
                                           `projectid` int(11) NOT NULL,
                                           `createtime` bigint(20) NOT NULL,
                                           `updatetime` bigint(20) NOT NULL,
                                           `serial_number` varchar(128) NOT NULL,
                                           PRIMARY KEY (`id`) USING BTREE,
                                           UNIQUE KEY `deviceid` (`deviceid`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=COMPACT;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备-项目关联表，记录设备与项目的归属关系。
