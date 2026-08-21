---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_pc_cfg

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

真机节点配置表：节点 IP、位置等。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_pc_cfg`  (
                                   `ucomid` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '上位机账号',
                                   `ip` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '上位机ip',
                                   `clouds` text CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '设备云',
                                   `pc_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '上位机配置信息',
                                   `net_manage` smallint(2) NULL DEFAULT NULL COMMENT '网络管理   0-自动;
```

## 索引

- DDL 中未见显式索引

## 被哪些接口/mapper 方法使用

- `RealcfgPcCfgDAOImpl`（JDBC）← `PcCfgServiceImpl` ← 接口 [PcCfg](../../平台配置（real-cfg）/07-开放接口文档/用户与权限/PcCfg.md)
- 被视图 [view_pc_account](view_pc_account.md) 引用
