---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL视图
---

# view_device_condition

## 用途

设备条件枚举视图（UNION ALL）。ViewDeviceConditionDAOImpl 查询使用，用于前端筛选条件下拉。

## 所属数据库

db_device

## DDL

```sql
CREATE ALGORITHM = UNDEFINED DEFINER = `root`@`%` SQL SECURITY DEFINER VIEW `view_device_condition` AS select `view_device_source_info`.`release_ver` AS `key_name`,`view_device_source_info`.`source` AS `source`,'releaseVer' AS `type` from `view_device_source_info` group by `view_device_source_info`.`source`,`key_name` union all select `view_device_source_info`.`view_resolution` AS `key_name`,`view_device_source_info`.`source` AS `source`,'resolution' AS `type` from `view_device_source_info` group by `view_device_source_info`.`source`,`key_name` union all select `view_device_source_info`.`region` AS `key_name`,`view_device_source_info`.`source` AS `source`,'region' AS `type` from `view_device_source_info` group by `view_device_source_info`.`source`,`key_name` union all select `view_device_source_info`.`syspf_name` AS `key_name`,`view_device_source_info`.`source` AS `source`,'syspfName' AS `type` from `view_device_source_info` group by `view_device_source_info`.`source`,`key_name` union all select `view_device_source_info`.`brand_name` AS `key_name`,`view_device_source_info`.`source` AS `source`,'brandName' AS `type` from `view_device_source_info` group by `view_device_source_info`.`source`,`key_name` union all select `view_device_source_info`.`model_name` AS `key_name`,`view_device_source_info`.`source` AS `source`,'modelName' AS `type` from `view_device_source_info` group by `view_device_source_info`.`source`,`key_name`;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备条件枚举视图（UNION ALL）。ViewDeviceConditionDAOImpl 查询使用，用于前端筛选条件下拉。
