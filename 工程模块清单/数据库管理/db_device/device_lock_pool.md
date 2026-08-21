---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_lock_pool

## 用途

设备锁定池表。TaskDeviceLockPoolMapper/MyBatis 操作。用于任务执行期间的设备锁定管理，防止多任务抢占同一设备。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_lock_pool` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT,
    `task_type` smallint(4) NOT NULL,
    `task_id` varchar(64) NOT NULL,
    `task_descr` varchar(64) DEFAULT NULL,
    `device_id` varchar(255) DEFAULT NULL,
    `ucom_id` varchar(255) DEFAULT NULL,
    `create_time` datetime DEFAULT CURRENT_TIMESTAMP,
    `update_time` datetime DEFAULT CURRENT_TIMESTAMP,
    `device_type` smallint(6) NOT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `uk_device_ucom` (`device_id`,`ucom_id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备锁定池表。TaskDeviceLockPoolMapper/MyBatis 操作。用于任务执行期间的设备锁定管理，防止多任务抢占同一设备。
