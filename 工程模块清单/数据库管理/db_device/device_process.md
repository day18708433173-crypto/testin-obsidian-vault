---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_process

## 用途

设备执行流程/任务记录表。DeviceDispatchThread 任务匹配后写入流程，DeviceWorkerThread 超时时清理关联流程。实时查询视图为 view_device_process。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_process`  (
                                   `id` bigint(20) NOT NULL AUTO_INCREMENT,
                                   `job_id` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
                                   `vhost` int(11) NOT NULL,
                                   `eid` int(11) NOT NULL,
                                   `projectid` int(11) NOT NULL COMMENT '0 默认值',
                                   `device_action` int(11) NOT NULL COMMENT '1 测试; 2 真机online 等; 3 第三方系统使用\n',
                                   `deviceid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                   `modelid` int(11) NOT NULL,
                                   `device_region` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                   `release_ver` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                   `serial_number` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                   `additional_content` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
                                   `assets_num` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                   `assets_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
                                   `userid` int(11) NOT NULL,
                                   `user_name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                   `user_email` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                   `user_region` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                   `user_dept` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                   `user_team` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                   `totaltime` bigint(20) NULL DEFAULT NULL,
                                   `descr` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                   `mark_name_1` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                   `mark_name_2` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                   `status` int(11) NOT NULL COMMENT '0 执行中 1 已完成',
                                   `createtime` bigint(20) NOT NULL,
                                   `updatetime` bigint(20) NOT NULL,
                                   PRIMARY KEY (`id`) USING BTREE,
                                   UNIQUE INDEX `job_id`(`job_id`) USING BTREE,
                                   INDEX `status_deviceid`(`status`, `deviceid`) USING BTREE,
                                   INDEX `status_vhost`(`status`, `vhost`) USING BTREE,
                                   INDEX `p_ct_da`(`projectid`, `createtime`, `device_action`) USING BTREE,
                                   INDEX `p_ct_ma`(`projectid`, `createtime`, `mark_name_1`, `mark_name_2`) USING BTREE,
                                   INDEX `p_ct_ur_ud_ut`(`projectid`, `createtime`, `user_region`, `user_dept`, `user_team`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备执行流程/任务记录表。DeviceDispatchThread 任务匹配后写入流程，DeviceWorkerThread 超时时清理关联流程。实时查询视图为 view_device_process。
