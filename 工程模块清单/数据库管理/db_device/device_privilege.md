---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_privilege

## 用途

设备使用权限表，记录设备对用户/项目的使用权限配置。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_privilege`  (
                                     `source` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                     `deviceid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                     `debug` int(11) NULL DEFAULT NULL COMMENT '调试',
                                     `simulate_network` int(11) NULL DEFAULT NULL COMMENT '模拟网络',
                                     `record_script` int(11) NULL DEFAULT NULL COMMENT '录制脚本',
                                     `regression` int(11) NULL DEFAULT NULL COMMENT '回归测试',
                                     `usability` int(11) NULL DEFAULT NULL COMMENT '可用性测试',
                                     `sale` int(11) NULL DEFAULT NULL COMMENT '可售卖',
                                     `remark` varchar(200) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                     `createtime` bigint(20) NULL DEFAULT NULL,
                                     `updatetime` bigint(20) NULL DEFAULT NULL,
                                     PRIMARY KEY (`source`, `deviceid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备使用权限表，记录设备对用户/项目的使用权限配置。
