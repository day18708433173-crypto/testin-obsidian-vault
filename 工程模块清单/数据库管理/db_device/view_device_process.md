---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL视图
---

# view_device_process

## 用途

设备流程-机型联合视图（device_process LEFT JOIN device_model_snapshot）。用于流程列表展示。

## 所属数据库

db_device

## DDL

```sql
CREATE ALGORITHM = UNDEFINED DEFINER = `root`@`%` SQL SECURITY DEFINER VIEW `view_device_process` AS select `device_process`.`id` AS `id`,`device_process`.`job_id` AS `job_id`,`device_process`.`vhost` AS `vhost`,`device_process`.`device_action` AS `device_action`,`device_process`.`deviceid` AS `deviceid`,`device_process`.`modelid` AS `modelid`,`device_process`.`device_region` AS `device_region`,`device_process`.`release_ver` AS `release_ver`,`device_process`.`eid` AS `eid`,`device_process`.`serial_number` AS `serial_number`,`device_process`.`additional_content` AS `additional_content`,`device_process`.`assets_num` AS `assets_num`,`device_process`.`assets_config` AS `assets_config`,`device_process`.`userid` AS `userid`,`device_process`.`projectid` AS `projectid`,`device_process`.`user_name` AS `user_name`,`device_process`.`user_email` AS `user_email`,`device_process`.`user_region` AS `user_region`,`device_process`.`totaltime` AS `totaltime`,`device_process`.`descr` AS `descr`,`device_process`.`status` AS `status`,`device_process`.`createtime` AS `createtime`,`device_process`.`updatetime` AS `updatetime`,`device_model_snapshot`.`brand_id` AS `brand_id`,`device_model_snapshot`.`brand_name` AS `brand_name`,`device_model_snapshot`.`brand_abbr` AS `brand_abbr`,`device_model_snapshot`.`model_name` AS `model_name`,`device_model_snapshot`.`model_alias` AS `model_alias`,`device_model_snapshot`.`model_type` AS `model_type`,`device_model_snapshot`.`os_name` AS `os_name`,`device_model_snapshot`.`dpi_width` AS `dpi_width`,`device_model_snapshot`.`dpi_height` AS `dpi_height`,`device_model_snapshot`.`screen_size` AS `screen_size`,`device_process`.`user_dept` AS `user_dept`,`device_process`.`user_team` AS `user_team`,`device_process`.`mark_name_1` AS `mark_name_1`,`device_process`.`mark_name_2` AS `mark_name_2` from (`device_process` left join `device_model_snapshot` on((`device_process`.`modelid` = `device_model_snapshot`.`modelid`)));
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备流程-机型联合视图（device_process LEFT JOIN device_model_snapshot）。用于流程列表展示。
