---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_total

## 用途

设备汇总统计表/视图。通过 AppDeviceActionLogDAO 使用 TABLE_NAME = "device_total"。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_total`  (
                                 `cloud` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '设备云信息',
                                 `device_status` int(11) NOT NULL COMMENT '设备状态信息,free = 0, runScript = 1, disconnect = 2, unknown = 9',
                                 `total` int(11) NOT NULL COMMENT '统计数量',
                                 `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                 `batch_no` bigint(20) NOT NULL COMMENT '批次号'
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备汇总统计表/视图。通过 AppDeviceActionLogDAO 使用 TABLE_NAME = "device_total"。
