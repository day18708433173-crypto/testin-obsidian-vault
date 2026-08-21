---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL视图
---

# view_pc_condition

## 用途

PC条件枚举视图。ViewPcConditionDAOImpl 查询使用，用于前端筛选。

## 所属数据库

db_pc

## DDL

```sql
CREATE VIEW `view_pc_condition` AS select `view_pc_info_source`.`type` AS `key_name`,`view_pc_info_source`.`source` AS `source`,'type' AS `type` from `view_pc_info_source` group by `view_pc_info_source`.`source`,`key_name` union all select `view_pc_info_source`.`version` AS `key_name`,`view_pc_info_source`.`source` AS `source`,'version' AS `type` from `view_pc_info_source` group by `view_pc_info_source`.`source`,`key_name` union all select `view_pc_info_source`.`os_name` AS `key_name`,`view_pc_info_source`.`source` AS `source`,'osName' AS `type` from `view_pc_info_source` group by `view_pc_info_source`.`source`,`key_name` union all select `view_pc_info_source`.`ip` AS `key_name`,`view_pc_info_source`.`source` AS `source`,'ip' AS `type` from `view_pc_info_source` group by `view_pc_info_source`.`source`,`key_name`;
```

> DDL 来源：pocinit/src/mysql/db_pc.sql（命中）

## 设备控制中心 中的使用

PC条件枚举视图。ViewPcConditionDAOImpl 查询使用，用于前端筛选。
