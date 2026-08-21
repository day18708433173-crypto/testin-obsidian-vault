---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_pc_info

## 用途

设备-PC关联信息表，记录移动设备通过哪个PC节点连接。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_pc_info`  (
                                   `device_id` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '' COMMENT '设备id',
                                   `support_chrome` tinyint(1) NULL DEFAULT NULL COMMENT '是否支持chrome',
                                   `support_firefox` tinyint(1) NULL DEFAULT NULL COMMENT '是否支持firefox',
                                   `disk_info` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '磁盘信息',
                                   `os_info` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '系统信息',
                                   `cpu_info` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'cpu信息',
                                   `mem_info` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '内存信息',
                                   `device_status` tinyint(1) NULL DEFAULT NULL COMMENT '设备状态,1-在线;0-离线',
                                   `ui_task_num` int(10) NULL DEFAULT NULL COMMENT 'ui任务个数',
                                   `interface_task_num` int(10) NULL DEFAULT NULL COMMENT '接口任务个数',
                                   `reporttime` bigint(20) NULL DEFAULT NULL COMMENT '上报时间',
                                   `port` int(11) NULL DEFAULT NULL COMMENT '端口',
                                   PRIMARY KEY (`device_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备-PC关联信息表，记录移动设备通过哪个PC节点连接。
