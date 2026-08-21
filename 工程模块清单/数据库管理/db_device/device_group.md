---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_group

## 用途

设备分组表。DeviceGroupStatusThread 每10秒检查心跳超时的分组并标记为离线。DeviceGroupHandlerThread 任务匹配时查询分组信息。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_group`  (
                                 `id` int(11) NOT NULL AUTO_INCREMENT,
                                 `deviceid` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '设备组id',
                                 `group_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '设备组名称',
                                 `ucomid` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '上位机id',
                                 `status` int(1) NULL DEFAULT NULL,
                                 `action` int(1) NULL DEFAULT NULL,
                                 `groupDisplay` text CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL COMMENT '屏幕配置',
                                 `groupDevice` text CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL,
                                 `marks` varchar(2024) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
                                 `descr` varchar(1024) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
                                 `createdTime` bigint(20) NULL DEFAULT NULL,
                                 `updatedTime` bigint(20) NULL DEFAULT NULL,
                                 `projectid` int(11) NOT NULL,
                                 PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备分组表。DeviceGroupStatusThread 每10秒检查心跳超时的分组并标记为离线。DeviceGroupHandlerThread 任务匹配时查询分组信息。
