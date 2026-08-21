---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_raspi_cfg

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

树莓派节点配置表。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_raspi_cfg`  (
                                      `rpiid` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '树莓派编码',
                                      `ucomid` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '上位机账号id信息',
                                      `app_version` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'app 版本',
                                      `mac` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '树莓派mac地址',
                                      `ip` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '树莓派ip信息',
                                      `devices` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT ' 树莓派连接设备信息',
                                      `location` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '位置',
                                      `status` int(11) NOT NULL COMMENT '状态,off = 0, on = 1',
                                      `createtime` bigint(20) NOT NULL COMMENT '建立时间',
                                      `updatetime` bigint(20) NOT NULL COMMENT '更新时间',
                                      `reporttime` bigint(20) NOT NULL,
                                      PRIMARY KEY (`rpiid`, `reporttime`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`rpiid`, `reporttime`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgRaspiCfgDAOImpl`（JDBC）← `RaspiCfgServiceImpl` ← 接口 [RaspiCfg](../../平台配置（real-cfg）/07-开放接口文档/设备与网络配置/RaspiCfg.md)
