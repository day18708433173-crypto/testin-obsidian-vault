---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_device_source

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

设备资源表：设备云中的设备资源（含 iCloud 名称、父设备、任务过期周期等）。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_device_source`  (
                                          `name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '' COMMENT '设备云标识',
                                          `icloud_name` varchar(250) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '设备云名称',
                                          `parent_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                          `descr` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '描述',
                                          `operater` int(11) NOT NULL COMMENT '云创建人',
                                          `task_expire_period` bigint(20) NULL DEFAULT NULL,
                                          `config` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '配置信息{\"privileges\":{}, \"other\":{}}',
                                          `status` int(11) NOT NULL COMMENT '状态,off = 0, on = 1',
                                          `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                          `updatetime` bigint(20) NOT NULL COMMENT '更新时间',
                                          PRIMARY KEY (`name`) USING BTREE,
                                          UNIQUE INDEX `icloud_name`(`icloud_name`, `parent_name`) USING BTREE,
                                          INDEX `parent_name`(`parent_name`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`name`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgDeviceSourceDAOImpl`（JDBC）← `DeviceSourceServiceImpl` ← 接口 [DeviceSource](../../平台配置（real-cfg）/07-开放接口文档/设备与网络配置/DeviceSource.md)
- 被视图 [view_filterproper_source](view_filterproper_source.md)、[view_project_group_source](view_project_group_source.md) 引用
