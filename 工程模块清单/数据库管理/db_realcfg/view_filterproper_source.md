---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# view_filterproper_source

- 数据库：`db_realcfg`
- 对象类型：视图

## 用途

未被任何项目组占用的设备资源视图（过滤 realcfg_project_group 已占用设备）。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE VIEW `view_filterproper_source` AS select `realcfg_device_source`.`name` AS `name`,`realcfg_device_source`.`icloud_name` AS `icloud_name`,`realcfg_device_source`.`parent_name` AS `parent_name`,`realcfg_device_source`.`descr` AS `descr`,`realcfg_device_source`.`operater` AS `operater`,`realcfg_device_source`.`status` AS `status`,`realcfg_device_source`.`createtime` AS `createtime`,`realcfg_device_source`.`updatetime` AS `updatetime`,`realcfg_device_source`.`task_expire_period` AS `task_expire_period`,`realcfg_device_source`.`config` AS `config` from `realcfg_device_source` where (isnull(`realcfg_device_source`.`parent_name`) and (not(`realcfg_device_source`.`name` in (select `realcfg_project_group`.`devicegroupid` from `realcfg_project_group`))));
```

## 索引

- 视图，无索引

## 被哪些接口/mapper 方法使用

- `RealcfgDeviceSourceDAOImpl`（FilterProper 查询）← `DeviceSourceServiceImpl` ← 接口 [DeviceSource](../../平台配置（real-cfg）/07-开放接口文档/设备与网络配置/DeviceSource.md)
