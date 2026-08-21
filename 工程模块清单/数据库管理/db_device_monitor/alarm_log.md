---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# alarm_log

## 用途

设备监控报警日志表。AlarmLogServiceScheduled 每分钟检查设备温度数据，超阈值时写入报警记录。AlarmLogMapper/MyBatis 和 JDBC 双重操作。

## 所属数据库

db_device_monitor

## DDL

```sql
CREATE TABLE `alarm_log`  (
                              `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '报警id',
                              `device_id` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '报警对象',
                              `device_brand` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '设备品牌',
                              `device_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '设备名称',
                              `model_alias` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
                              `rule_id` int(11) NOT NULL COMMENT '规则id',
                              `event` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '事件',
                              `alarm_value` double(4, 1) NOT NULL COMMENT '报警时数值',
                              `alarm_time` datetime(0) NOT NULL COMMENT '报警时间',
                              `create_time` datetime(0) NOT NULL COMMENT '创建事件',
                              `update_time` datetime(0) NOT NULL COMMENT '更新时间',
                              `status` tinyint(1) NOT NULL DEFAULT 0 COMMENT '状态，1是已读，0是未读',
                              `deleted` tinyint(1) NULL DEFAULT 0 COMMENT '1是删除,0是未删除',
                              PRIMARY KEY (`id`) USING BTREE,
                              INDEX `idx_status_deleted`(`status`, `deleted`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device_monitor.sql（命中）

## 设备控制中心 中的使用

设备监控报警日志表。AlarmLogServiceScheduled 每分钟检查设备温度数据，超阈值时写入报警记录。AlarmLogMapper/MyBatis 和 JDBC 双重操作。
