---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_detail

## 用途

设备详情表。DeviceDetailMapper/MyBatis 操作。存储设备的详细描述信息。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_detail` (
    `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `device_id` varchar(255) DEFAULT NULL COMMENT '设备id',
    `type` tinyint(1) DEFAULT NULL COMMENT '设备类型 1-app  3-web  5-pc',
    `visible_status` tinyint(1) DEFAULT '1' COMMENT '可见范围  1-所有项目  2-项目独占',
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

设备详情表。DeviceDetailMapper/MyBatis 操作。存储设备的详细描述信息。
