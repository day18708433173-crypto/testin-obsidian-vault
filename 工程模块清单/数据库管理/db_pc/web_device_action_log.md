---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# web_device_action_log

## 用途

Web设备操作日志表。WebDeviceActionLogDAO/JDBC 和 WebDeviceActionLogMapper/MyBatis 操作。记录浏览器控制设备的操作动作。TABLE_NAME = "db_pc.web_device_action_log"。

## 所属数据库

db_pc

## DDL

```sql
CREATE TABLE `web_device_action_log`  (
                                          `id` int(11) NOT NULL AUTO_INCREMENT,
                                          `device_ip` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '设备ip',
                                          `status` tinyint(1) NULL DEFAULT NULL COMMENT '状态',
                                          `create_time` bigint(20) NULL DEFAULT NULL COMMENT '创建时间',
                                          `action` tinyint(1) NULL DEFAULT NULL COMMENT '操作类型',
                                          PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_pc.sql（命中）

## 设备控制中心 中的使用

Web设备操作日志表。WebDeviceActionLogDAO/JDBC 和 WebDeviceActionLogMapper/MyBatis 操作。记录浏览器控制设备的操作动作。TABLE_NAME = "db_pc.web_device_action_log"。
