---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# view_pc_account

- 数据库：`db_realcfg`
- 对象类型：视图

## 用途

真机账号-节点配置视图：realcfg_pc_account LEFT JOIN realcfg_pc_cfg。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE VIEW `view_pc_account` AS select `realcfg_pc_account`.`ucomid` AS `ucomid`,`realcfg_pc_account`.`ucomid_pwd` AS `ucomid_pwd`,`realcfg_pc_account`.`descr` AS `descr`,`realcfg_pc_account`.`sign` AS `sign`,`realcfg_pc_account`.`signvhost` AS `signvhost`,`realcfg_pc_account`.`signtime` AS `signtime`,`realcfg_pc_account`.`signouttime` AS `signouttime`,`realcfg_pc_account`.`status` AS `status`,`realcfg_pc_account`.`createtime` AS `createtime`,`realcfg_pc_account`.`updatetime` AS `updatetime`,`realcfg_pc_account`.`lastaccesstime` AS `lastaccesstime`,`realcfg_pc_cfg`.`ip` AS `ip`,`realcfg_pc_cfg`.`location` AS `location` from (`realcfg_pc_account` left join `realcfg_pc_cfg` on((`realcfg_pc_account`.`ucomid` = `realcfg_pc_cfg`.`ucomid`)));
```

## 索引

- 视图，无索引

## 被哪些接口/mapper 方法使用

- `RealcfgPcAccountDAOImpl`（pojo `ViewPcAccount`）← `PcAccountServiceImpl` ← 接口 [PcAccount](../../平台配置（real-cfg）/07-开放接口文档/用户与权限/PcAccount.md)
