---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_assets_info

## 用途

设备资产信息表（机柜位置、区域、资产编号、附加配置等）。关联到 view_device_info 和 view_device_source_info 视图中。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_assets_info`  (
                                       `deviceid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '设备id',
                                       `assets_num` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '资产编号',
                                       `region` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                       `owner` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '拥有者',
                                       `descr` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '描述',
                                       `operator` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '操作者',
                                       `additional_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
                                       `status` int(11) NOT NULL,
                                       `createtime` bigint(20) NOT NULL,
                                       `updatetime` bigint(20) NOT NULL,
                                       `cabin_site` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '机柜中设备位置',
                                       `mark1` varchar(1024) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '备用字段1',
                                       `mark2` varchar(1024) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '备用字段2',
                                       PRIMARY KEY (`deviceid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备资产信息表（机柜位置、区域、资产编号、附加配置等）。关联到 view_device_info 和 view_device_source_info 视图中。
