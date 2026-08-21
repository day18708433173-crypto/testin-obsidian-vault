---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_certificate

## 用途

设备-证书关联表，记录设备安装了哪些证书。DeviceWorkerThread 清理时会通过 DeviceCertificateDAO 清理关联。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_certificate`  (
                                       `deviceid` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '设备id',
                                       `certificate_id` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '证书id'
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备-证书关联表，记录设备安装了哪些证书。DeviceWorkerThread 清理时会通过 DeviceCertificateDAO 清理关联。
