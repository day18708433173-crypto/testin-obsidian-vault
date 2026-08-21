---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_project_exclusive

## 用途

设备项目独占配置表。DeviceProjectExclusiveMapper/MyBatis 操作。配置设备对特定项目的独占使用。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_project_exclusive` (
    `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `device_id` varchar(255) DEFAULT NULL COMMENT '设备id',
    `type` tinyint(4) DEFAULT NULL COMMENT '设备类型',
    `project_exclusive_id` int(11) DEFAULT NULL COMMENT '独占项目id',
    `create_user_id` int(11) DEFAULT NULL COMMENT '创建人',
    `create_time` datetime DEFAULT NULL COMMENT '创建时间',
    `update_user_id` int(11) DEFAULT NULL COMMENT '更新人',
    `update_time` datetime DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (`id`),
    KEY `idx_device_id` (`device_id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备项目独占配置表。DeviceProjectExclusiveMapper/MyBatis 操作。配置设备对特定项目的独占使用。
