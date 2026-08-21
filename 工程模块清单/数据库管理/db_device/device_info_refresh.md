---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_info_refresh

## 用途

设备信息刷新辅助表，用于中间态暂存设备刷新数据。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_info_refresh`  (
                                        `pool_id` int(11) NOT NULL,
                                        `deviceid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                        `vhost` int(11) NOT NULL,
                                        `ext_deviceid` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                        `ucomid` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                        `modelid` int(11) NULL DEFAULT NULL,
                                        `status` int(11) NULL DEFAULT NULL,
                                        `action` int(11) NULL DEFAULT NULL,
                                        `debug_mode` int(11) NULL DEFAULT NULL,
                                        `debug_owner` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                        `network_type` int(11) NULL DEFAULT NULL,
                                        `network` int(11) NULL DEFAULT NULL,
                                        `wifi_ssid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                        `ram_avail` bigint(20) NULL DEFAULT NULL,
                                        `rom_avail` bigint(20) NULL DEFAULT NULL,
                                        `sdcard_avail` bigint(20) NULL DEFAULT NULL,
                                        `sdcard_writeable` int(11) NULL DEFAULT NULL,
                                        `battery_level` int(11) NULL DEFAULT NULL,
                                        `root_enable` int(11) NULL DEFAULT NULL,
                                        `minicap_enable` int(11) NULL DEFAULT NULL,
                                        `error_count` int(11) NULL DEFAULT NULL,
                                        `error_msg` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                        `licences` int(11) NULL DEFAULT NULL,
                                        `networktime` bigint(20) NULL DEFAULT NULL,
                                        `devicetime` bigint(20) NULL DEFAULT NULL,
                                        `refreshtime` bigint(20) NULL DEFAULT NULL,
                                        `rpiid` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                        `location` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                        `debug_port` int(11) NULL DEFAULT NULL,
                                        `source_rule` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                        `certificate_status` int(11) NULL DEFAULT NULL,
                                        `temperature` double NULL DEFAULT NULL,
                                        `machine_version` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '车机版本',
                                        `mcu_version` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '车机mcu版本',
                                        `icc_id` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'ICCID',
                                        `rom_update_status` int(11) NULL DEFAULT NULL COMMENT '0没有升级1下载中2上传设备中3升级中4升级完成',
                                        `ext_info` varchar(1024) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '扩展信息',
                                        `current_applicant` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '当前预约者',
                                        PRIMARY KEY (`pool_id`, `deviceid`) USING HASH
) ENGINE = MEMORY CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Fixed;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备信息刷新辅助表，用于中间态暂存设备刷新数据。
