---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# app_device_action_log

## 用途

App设备操作日志表。通过 AppDeviceActionLogDAO/JDBC 和 AppDeviceActionLogMapper/MyBatis 读写。记录设备操作动作、IP、时间等。TABLE_NAME = "db_device.app_device_action_log"。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `app_device_action_log`  (
                                          `id` int(11) NOT NULL AUTO_INCREMENT,
                                          `device_id` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '设备id',
                                          `model_id` int(11) DEFAULT NULL COMMENT '型号id',
                                          `device_brand_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '品牌名',
                                          `device_model_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '型号名',
                                          `status` tinyint(1) NULL DEFAULT NULL COMMENT '状态(0空闲 1执行脚本 2断开 9未知)',
                                          `action` tinyint(1) NULL DEFAULT NULL COMMENT '操作类型',
                                          `create_time` bigint(20) NULL DEFAULT NULL COMMENT '创建时间',
                                          PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 84 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

App设备操作日志表。通过 AppDeviceActionLogDAO/JDBC 和 AppDeviceActionLogMapper/MyBatis 读写。记录设备操作动作、IP、时间等。TABLE_NAME = "db_device.app_device_action_log"。
