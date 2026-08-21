---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_info_source

## 用途

设备来源/资源标记表（source、debug、simulate_network、record_script、regression等标记）。DeviceCountThread 通过 IDeviceInfoSourceDAO.devices() 统计设备数。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_info_source`  (
                                       `deviceid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                       `source` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                       `debug` int(11) NULL DEFAULT NULL,
                                       `simulate_network` int(11) NULL DEFAULT NULL,
                                       `record_script` int(11) NULL DEFAULT NULL,
                                       `regression` int(11) NULL DEFAULT NULL,
                                       `usability` int(11) NULL DEFAULT NULL,
                                       `sale` int(11) NULL DEFAULT NULL,
                                       `createtime` bigint(20) NOT NULL,
                                       `updatetime` bigint(20) NULL DEFAULT NULL,
                                       PRIMARY KEY (`deviceid`, `source`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备来源/资源标记表（source、debug、simulate_network、record_script、regression等标记）。DeviceCountThread 通过 IDeviceInfoSourceDAO.devices() 统计设备数。
