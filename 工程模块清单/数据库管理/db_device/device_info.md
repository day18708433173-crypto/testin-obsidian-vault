---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_info

## 用途

设备主信息表。设备上线时 INSERT/UPDATE，设备离线/超时时更新状态。通过 AbstractGenericDaoImpl.getDevicedao() 访问。DeviceWorkerThread 周期性检测超时设备并下线。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_info`  (
                                `vhost` int(11) NOT NULL,
                                `deviceid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '设备ID',
                                `ext_deviceid` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `ucomid` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '上位机账号',
                                `modelid` int(11) NULL DEFAULT NULL,
                                `device_model_id` int(11) NOT NULL COMMENT '设备机型id信息',
                                `device_brand_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '上报的品牌系列名称',
                                `device_model_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '上报设备机型名称',
                                `serial_number` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '人工调整sn信息',
                                `syspf_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'android/ios',
                                `release_ver` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '设备版本信息',
                                `sdk_ver` int(11) NULL DEFAULT NULL COMMENT 'SDK版本',
                                `base_imei` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'imei 信息',
                                `base_androidid` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'androidid 信息',
                                `base_mac` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'mac 信息',
                                `base_serial_number` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '上报的sn信息',
                                `action` int(11) NULL DEFAULT NULL COMMENT '动作： 0 空闲; 1 测试; 2真机调试',
                                `rpiid` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '树莓派id信息',
                                `debug_mode` int(11) NULL DEFAULT NULL COMMENT '是否支持真机调试',
                                `debug_owner` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `dpi_height` int(11) NULL DEFAULT NULL,
                                `dpi_width` int(11) NULL DEFAULT NULL,
                                `ram_total` bigint(20) NULL DEFAULT NULL,
                                `ram_avail` bigint(20) NULL DEFAULT NULL,
                                `rom_total` bigint(20) NULL DEFAULT NULL,
                                `rom_avail` bigint(20) NULL DEFAULT NULL,
                                `sdcard_total` bigint(20) NULL DEFAULT NULL,
                                `sdcard_avail` bigint(20) NULL DEFAULT NULL,
                                `sdcard_writeable` int(11) NULL DEFAULT NULL,
                                `cpu_brand` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `cpu_model` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `cpu_processor` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `cpu_freq` int(11) NULL DEFAULT NULL,
                                `cpu_num` int(11) NULL DEFAULT NULL,
                                `gpu_vendor` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `gpu_renderer` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `gpu_version` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `bluetooth` int(11) NULL DEFAULT NULL,
                                `bluetooth_version` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `nfc` int(11) NULL DEFAULT NULL,
                                `kernel_version` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
                                `build_version` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
                                `user_agent` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
                                `phone_number` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `network_type` int(11) NOT NULL COMMENT '网络类型： wifi、mobile',
                                `network` int(11) NOT NULL,
                                `wifi_ssid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `battery_level` int(11) NULL DEFAULT NULL,
                                `root_enable` int(11) NULL DEFAULT NULL,
                                `minicap_enable` int(11) NULL DEFAULT NULL,
                                `error_count` int(11) NULL DEFAULT NULL,
                                `error_msg` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `status` int(11) NOT NULL,
                                `licences` int(11) NULL DEFAULT 1 COMMENT '许可',
                                `createtime` bigint(20) NOT NULL,
                                `updatetime` bigint(20) NOT NULL,
                                `networktime` bigint(20) NULL DEFAULT NULL,
                                `devicetime` bigint(20) NULL DEFAULT NULL,
                                `refreshtime` bigint(20) NOT NULL,
                                `location` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `debug_port` int(11) NULL DEFAULT NULL,
                                `source_rule` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `lock_expiretime` bigint(20) NULL DEFAULT 0,
                                `certificate_status` int(11) NULL DEFAULT 0,
                                `temperature` double NULL DEFAULT NULL,
                                `machine_version` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '车机版本',
                                `mcu_version` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '车机mcu版本',
                                `icc_id` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'ICCID',
                                `rom_update_status` int(11) NULL DEFAULT NULL COMMENT '0没有升级1下载中2上传设备中3升级中4升级完成',
                                `test_mode` int(11) NULL DEFAULT 1 COMMENT '1开启提测 0关闭提测',
                                `ext_info` varchar(1024) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '扩展信息',
                                `current_applicant` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '当前预约者',
                                `screen_mode` int(1) NULL DEFAULT 0 COMMENT '屏幕状态 0不息屏 1息屏 默认是0',
                                PRIMARY KEY (`deviceid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备主信息表。设备上线时 INSERT/UPDATE，设备离线/超时时更新状态。通过 AbstractGenericDaoImpl.getDevicedao() 访问。DeviceWorkerThread 周期性检测超时设备并下线。
