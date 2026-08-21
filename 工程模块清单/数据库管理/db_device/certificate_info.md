---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# certificate_info

## 用途

iOS/Android 证书信息表（证书ID、名称、类型、包名、到期时间等）。DeviceWorkerThread 超时清理设备时会同步清理关联证书。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `certificate_info`  (
                                     `certificate_id` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '证书的id',
                                     `name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '证书名称',
                                     `type` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '类型',
                                     `appid` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '应用id',
                                     `package_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '包名',
                                     `certificate_num` int(11) NULL DEFAULT NULL COMMENT '证书数量',
                                     `descr` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '描述',
                                     `certificate_identifier` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '证书标识符',
                                     `expiretime` bigint(20) NULL DEFAULT NULL COMMENT '到期时间',
                                     `fileid` bigint(20) NULL DEFAULT NULL COMMENT '文件id',
                                     `file_md5` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '文件md5',
                                     `file_url` varchar(300) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '文件地址',
                                     `status` int(11) NOT NULL COMMENT '状态，0无效，1有效',
                                     `createtime` bigint(20) NOT NULL,
                                     `updatetime` bigint(20) NOT NULL,
                                     PRIMARY KEY (`certificate_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

iOS/Android 证书信息表（证书ID、名称、类型、包名、到期时间等）。DeviceWorkerThread 超时清理设备时会同步清理关联证书。
