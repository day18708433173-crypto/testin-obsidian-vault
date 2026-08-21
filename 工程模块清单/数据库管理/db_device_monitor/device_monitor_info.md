---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_monitor_info

## 用途

设备监控信息表。DeviceMonitorInfoMapper/MyBatis 操作。存储设备监控配置和监控数据。

## 所属数据库

db_device_monitor

## DDL

```sql
CREATE TABLE `device_monitor_info`  (
                                        `rule_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '规则id',
                                        `rule_name` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '规则名称',
                                        `duration` int(1) NOT NULL COMMENT '时间周期（分钟数）',
                                        `target` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '监控对象',
                                        `threshold_condition` int(255) NOT NULL COMMENT '阈值条件',
                                        `threshold_condition_symbol` int(255) NOT NULL COMMENT '阈值符号',
                                        `threshold_value` double(4, 1) NOT NULL COMMENT '阈值数值',
                                        `trigger_times` int(255) NOT NULL COMMENT '触发条件',
                                        `interval_cycle` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '沉默通道周期',
                                        `start_time` time(0) NOT NULL COMMENT '生效开始时间',
                                        `end_time` time(0) NOT NULL COMMENT '生效结束时间',
                                        `is_page_notice` tinyint(1) NULL DEFAULT NULL COMMENT '是否页面通知',
                                        `is_email_notice` tinyint(1) NULL DEFAULT NULL COMMENT '是否邮件通知',
                                        `email_address` text CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL COMMENT '邮件地址',
                                        `email_descr` text CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL COMMENT '邮件描述',
                                        `create_user_id` int(11) NOT NULL COMMENT '创建人id',
                                        `create_time` datetime(0) NOT NULL COMMENT '规则创建时间',
                                        `update_time` datetime(0) NOT NULL COMMENT '规则修改时间',
                                        `status` tinyint(1) NOT NULL COMMENT '状态',
                                        `rule_type` int(11) NOT NULL,
                                        PRIMARY KEY (`rule_id`) USING BTREE,
                                        INDEX `rule_name`(`rule_name`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device_monitor.sql（命中）

## 设备控制中心 中的使用

设备监控信息表。DeviceMonitorInfoMapper/MyBatis 操作。存储设备监控配置和监控数据。
