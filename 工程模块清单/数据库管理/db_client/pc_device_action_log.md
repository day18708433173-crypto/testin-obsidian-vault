---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# pc_device_action_log

## 用途

PC设备操作日志表（存储在 db_client 库）。PcDeviceActionLogDAO/JDBC 和 PcDeviceActionLogMapper/MyBatis 操作。记录PC控制手机设备的操作动作。TABLE_NAME = "db_client.pc_device_action_log"。

## 所属数据库

db_client

## DDL

```sql
CREATE TABLE `pc_device_action_log`  (
                                         `id` int(11) NOT NULL AUTO_INCREMENT,
                                         `device_ip` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '设备id',
                                         `status` tinyint(1) NULL DEFAULT NULL COMMENT '状态',
                                         `create_time` bigint(20) NULL DEFAULT NULL COMMENT '创建时间',
                                         `action` tinyint(1) NULL DEFAULT NULL COMMENT '操作类型',
                                         PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_client.sql（命中）

## 设备控制中心 中的使用

PC设备操作日志表（存储在 db_client 库）。PcDeviceActionLogDAO/JDBC 和 PcDeviceActionLogMapper/MyBatis 操作。记录PC控制手机设备的操作动作。TABLE_NAME = "db_client.pc_device_action_log"。
