---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL视图
---

# view_pc_info_source

## 用途

PC来源联合视图（pc_info_source JOIN pc_info JOIN pc_browser_info）。ViewPcInfoSourceDAOImpl 和 ViewPcInfoSourceMapper 查询使用。

## 所属数据库

db_pc

## DDL

```sql
CREATE VIEW `view_pc_info_source` AS select `pc_browser_info`.`type` AS `type`, `pc_browser_info`.`version` AS `version`, `pc_info`.`ucomid` AS `ucomid`, `pc_info`.`ip` AS `ip`, `pc_info`.`location` AS `location`, `pc_info`.`os_name` AS `os_name`, `pc_info`.`os_version` AS `os_version`, `pc_info`.`action` AS `action`, `pc_info`.`debug_owner` AS `debug_owner`, `pc_info`.`browsers` AS `browsers`, `pc_info`.`status` AS `status`, `pc_info`.`licences` AS `licences`, `pc_info`.`marks` AS `marks`, `pc_info`.`protocol` AS `protocol`, `pc_info`.`createtime` AS `createtime`, `pc_info`.`updatetime` AS `updatetime`, `pc_info_source`.`source` AS `source` from ((`pc_info_source` join `pc_info` on((`pc_info_source`.`ucomid` = `pc_info`.`ucomid`))) join `pc_browser_info` on((`pc_info`.`ucomid` = `pc_browser_info`.`ucomid`)));
```

> DDL 来源：pocinit/src/mysql/db_pc.sql（命中）

## 设备控制中心 中的使用

PC来源联合视图（pc_info_source JOIN pc_info JOIN pc_browser_info）。ViewPcInfoSourceDAOImpl 和 ViewPcInfoSourceMapper 查询使用。
