---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_device_cfg

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

设备配置表：设备级参数配置。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_device_cfg`  (
                                       `deviceid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '设备id',
                                       `clouds` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '设备云',
                                       `subclouds` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
                                       `descr` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '描述',
                                       `status` int(11) NOT NULL COMMENT '数据状态,off = 0, on = 1',
                                       `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                       `updatetime` bigint(20) NOT NULL COMMENT '更新时间',
                                       PRIMARY KEY (`deviceid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`deviceid`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgDeviceCfgDAOImpl`（JDBC）← `DeviceCfgServiceImpl` ← 接口 [DeviceCfg](../../平台配置（real-cfg）/07-开放接口文档/设备与网络配置/DeviceCfg.md)
