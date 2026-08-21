---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# view_project_group_source

- 数据库：`db_realcfg`
- 对象类型：视图

## 用途

项目组-设备资源视图：realcfg_device_source JOIN realcfg_project_group。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE VIEW `view_project_group_source` AS select `realcfg_project_group`.`eid` AS `eid`,`realcfg_project_group`.`projectid` AS `projectid`,`realcfg_device_source`.`name` AS `name`,`realcfg_device_source`.`icloud_name` AS `icloud_name`,`realcfg_device_source`.`parent_name` AS `parent_name`,`realcfg_device_source`.`descr` AS `descr`,`realcfg_device_source`.`operater` AS `operater`,`realcfg_device_source`.`status` AS `status`,`realcfg_device_source`.`createtime` AS `createtime`,`realcfg_device_source`.`updatetime` AS `updatetime`,`realcfg_project_group`.`expiretime` AS `expiretime`,`realcfg_project_group`.`status` AS `project_group_status`,`realcfg_device_source`.`config` AS `config`,`realcfg_device_source`.`task_expire_period` AS `task_expire_period` from (`realcfg_device_source` join `realcfg_project_group` on((`realcfg_device_source`.`name` = `realcfg_project_group`.`devicegroupid`)));
```

## 索引

- 视图，无索引

## 被哪些接口/mapper 方法使用

- `RealcfgDeviceSourceDAOImpl`（项目组设备查询）← `DeviceSourceServiceImpl` ← 接口 [DeviceSource](../../平台配置（real-cfg）/07-开放接口文档/设备与网络配置/DeviceSource.md)
