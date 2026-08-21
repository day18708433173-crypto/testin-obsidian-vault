---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_process_condition

## 用途

设备流程条件配置表，存储任务执行的条件约束。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_process_condition`  (
                                             `id` int(11) NOT NULL AUTO_INCREMENT,
                                             `type` int(11) NOT NULL COMMENT '1: 使用记录； 2：设备数量统计。',
                                             `name` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                             `eid` int(11) NOT NULL,
                                             `userid` int(11) NOT NULL,
                                             `content` varchar(2000) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                             `status` int(11) NOT NULL,
                                             `createtime` bigint(20) NOT NULL,
                                             `updatetime` bigint(20) NOT NULL,
                                             PRIMARY KEY (`id`) USING BTREE,
                                             INDEX `eid_user_type`(`eid`, `userid`, `type`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备流程条件配置表，存储任务执行的条件约束。
