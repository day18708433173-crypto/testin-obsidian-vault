---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL视图
---

# view_device_status_total

## 用途

设备状态汇总视图（按source+status GROUP BY count）。用于仪表盘展示设备状态分布。

## 所属数据库

db_device

## DDL

```sql
CREATE ALGORITHM = UNDEFINED DEFINER = `root`@`%` SQL SECURITY DEFINER VIEW `view_device_status_total` AS select `device_info_source`.`source` AS `source`,`device_info`.`status` AS `status`,count(1) AS `total` from (`device_info` join `device_info_source` on((`device_info_source`.`deviceid` = `device_info`.`deviceid`))) group by `device_info_source`.`source`,`device_info`.`status`;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备状态汇总视图（按source+status GROUP BY count）。用于仪表盘展示设备状态分布。
