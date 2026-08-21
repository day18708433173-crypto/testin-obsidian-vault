---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_appointment_config

## 用途

设备预约配置表。AppointmentConfigInitThread 每10分钟调用 loadConfigList() 刷新配置。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_appointment_config`  (
                                              `eid` int(11) NOT NULL COMMENT '企业ID：0 为默认配置项',
                                              `name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '配置名称',
                                              `descr` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '配置描述',
                                              `content_type` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '内容类型\n',
                                              `content` varchar(1000) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '配置内容',
                                              `group_info` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '分组信息',
                                              `status` int(11) NOT NULL COMMENT '状态信息',
                                              `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                              `updatetime` bigint(20) NOT NULL COMMENT '更新时间',
                                              UNIQUE INDEX `eid`(`eid`, `name`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备预约配置表。AppointmentConfigInitThread 每10分钟调用 loadConfigList() 刷新配置。
