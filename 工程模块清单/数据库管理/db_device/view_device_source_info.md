---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL视图
---

# view_device_source_info

## 用途

设备来源-机型联合视图（view_device_info基础上+device_info_source）。ViewDeviceSourceInfoDAOImpl 查询使用。是设备查询的主视图，包含 source/debug/simulate_network 等标记。

## 所属数据库

db_device

## DDL

```sql
CREATE ALGORITHM = UNDEFINED DEFINER = `root`@`%` SQL SECURITY DEFINER VIEW `view_device_source_info` AS select `device_info`.`deviceid` AS `deviceid`,`device_info`.`ucomid` AS `ucomid`,`device_info`.`device_model_id` AS `device_model_id`,`device_info`.`device_brand_name` AS `device_brand_name`,`device_info`.`device_model_name` AS `device_model_name`,`device_info`.`serial_number` AS `serial_number`,`device_info`.`syspf_name` AS `syspf_name`,`device_info`.`release_ver` AS `release_ver`,`device_info`.`sdk_ver` AS `sdk_ver`,`device_info`.`base_imei` AS `base_imei`,`device_info`.`base_androidid` AS `base_androidid`,`device_info`.`base_mac` AS `base_mac`,`device_info`.`base_serial_number` AS `base_serial_number`,`device_info`.`action` AS `action`,`device_info`.`rpiid` AS `rpiid`,`device_info`.`debug_mode` AS `debug_mode`,`device_info`.`ram_avail` AS `ram_avail`,`device_info`.`rom_avail` AS `rom_avail`,`device_info`.`sdcard_total` AS `sdcard_total`,`device_info`.`sdcard_avail` AS `sdcard_avail`,`device_info`.`sdcard_writeable` AS `sdcard_writeable`,`device_info`.`build_version` AS `build_version`,`device_info`.`kernel_version` AS `kernel_version`,`device_info`.`user_agent` AS `user_agent`,`device_info`.`phone_number` AS `phone_number`,`device_info`.`network_type` AS `network_type`,`device_info`.`network` AS `network`,`device_info`.`wifi_ssid` AS `wifi_ssid`,`device_info`.`battery_level` AS `battery_level`,`device_info`.`root_enable` AS `root_enable`,`device_info`.`minicap_enable` AS `minicap_enable`,`device_info`.`error_count` AS `error_count`,`device_info`.`error_msg` AS `error_msg`,`device_info`.`status` AS `status`,`device_info`.`licences` AS `licences`,`device_info`.`createtime` AS `createtime`,`device_info`.`updatetime` AS `updatetime`,`device_info`.`networktime` AS `networktime`,`device_info`.`devicetime` AS `devicetime`,`device_info`.`refreshtime` AS `refreshtime`,`device_model_snapshot`.`modelid` AS `modelid`,`device_model_snapshot`.`brand_id` AS `brand_id`,`device_model_snapshot`.`brand_name` AS `brand_name`,`device_model_snapshot`.`brand_abbr` AS `brand_abbr`,`device_model_snapshot`.`logo_url` AS `logo_url`,`device_model_snapshot`.`model_name` AS `model_name`,`device_model_snapshot`.`model_alias` AS `model_alias`,`device_model_snapshot`.`model_type` AS `model_type`,`device_model_snapshot`.`os_name` AS `os_name`,`device_model_snapshot`.`dpi_width` AS `dpi_width`,`device_model_snapshot`.`dpi_height` AS `dpi_height`,`device_model_snapshot`.`screen_size` AS `screen_size`,`device_model_snapshot`.`pic_url` AS `pic_url`,`device_model_snapshot`.`pic_url_m` AS `pic_url_m`,`device_model_snapshot`.`pic_url_b` AS `pic_url_b`,`device_model_snapshot`.`cpu_freq` AS `cpu_freq`,`device_model_snapshot`.`cpu_num` AS `cpu_num`,`device_model_snapshot`.`cpu_model` AS `cpu_model`,`device_model_snapshot`.`cpu_brand` AS `cpu_brand`,`device_model_snapshot`.`cpu_processor` AS `cpu_processor`,`device_model_snapshot`.`gpu_vendor` AS `gpu_vendor`,`device_model_snapshot`.`gpu_renderer` AS `gpu_renderer`,`device_model_snapshot`.`gpu_version` AS `gpu_version`,`device_model_snapshot`.`ram` AS `ram`,`device_model_snapshot`.`rom` AS `rom`,`device_model_snapshot`.`nfc` AS `nfc`,`device_model_snapshot`.`bluetooth` AS `bluetooth`,`device_model_snapshot`.`bluetooth_version` AS `bluetooth_version`,`device_model_snapshot`.`on_shelve_time` AS `on_shelve_time`,`device_model_snapshot`.`off_shelve_time` AS `off_shelve_time`,`device_model_snapshot`.`model_version` AS `model_version`,`device_model_snapshot`.`fingermark` AS `fingermark`,`device_model_snapshot`.`otg` AS `otg`,round((`device_model_snapshot`.`cpu_freq` / 1024),1) AS `view_cpu`,round((`device_model_snapshot`.`ram` / 1024),1) AS `view_ram`,concat_ws('*',`device_model_snapshot`.`dpi_width`,`device_model_snapshot`.`dpi_height`) AS `view_resolution`,`device_info`.`location` AS `location`,`device_assets_info`.`descr` AS `location_descr`,`device_info`.`debug_owner` AS `debug_owner`,`device_info`.`vhost` AS `vhost`,`device_info`.`source_rule` AS `source_rule`,`device_info`.`debug_port` AS `debug_port`,`device_info_source`.`source` AS `source`,`device_info_source`.`debug` AS `debug`,`device_info_source`.`simulate_network` AS `simulate_network`,`device_info_source`.`record_script` AS `record_script`,`device_info_source`.`regression` AS `regression`,`device_info_source`.`usability` AS `usability`,`device_info_source`.`sale` AS `sale`,`device_info`.`lock_expiretime` AS `lock_expiretime`,`device_info`.`certificate_status` AS `certificate_status`,`device_info`.`temperature` AS `temperature`,`device_model_snapshot`.`weight` AS `weight`,`device_assets_info`.`assets_num` AS `assets_num`,`device_assets_info`.`region` AS `region`,`device_info`.`mcu_version` AS `mcu_version`,`device_info`.`machine_version` AS `machine_version`,`device_info`.`icc_id` AS `icc_id`,`device_info`.`rom_update_status` AS `rom_update_status`,`device_info`.`test_mode` AS `test_mode`,`device_info`.`ext_info` AS `ext_info`,`device_inf
-- ... (truncated)
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备来源-机型联合视图（view_device_info基础上+device_info_source）。ViewDeviceSourceInfoDAOImpl 查询使用。是设备查询的主视图，包含 source/debug/simulate_network 等标记。
