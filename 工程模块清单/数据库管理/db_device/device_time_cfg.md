---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_time_cfg

## 用途

设备时间配置表。DeviceTimeCfgMapper/MyBatis 操作。存储设备可用时间段配置。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_time_cfg` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `device_id` varchar(255) DEFAULT NULL COMMENT '设备id',
    `type` tinyint(1) DEFAULT NULL COMMENT '设备类型 1-app 3-web  5-pc',
    `project_exclusive_id` int(11) DEFAULT NULL COMMENT '独占项目的id',
    `start_week` tinyint(1) DEFAULT NULL COMMENT '开始可用的周期  星期几',
    `start_time` time DEFAULT NULL COMMENT '开始可用的周期  时间点',
    `end_week` tinyint(1) DEFAULT NULL COMMENT '结束可用的周期  星期几',
    `end_time` time DEFAULT NULL COMMENT '结束可用的周期  时间点',
    `create_user_id` int(11) DEFAULT NULL COMMENT '创建人id',
    `update_user_id` int(11) DEFAULT NULL COMMENT '更新人id',
    `create_time` datetime DEFAULT NULL COMMENT '创建时间',
    `update_time` datetime DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (`id`),
    KEY `idx_device_id` (`device_id`) USING BTREE
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备时间配置表。DeviceTimeCfgMapper/MyBatis 操作。存储设备可用时间段配置。
