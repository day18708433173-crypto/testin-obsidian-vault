---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# ucom_info

## 用途

通用通信模块信息表（UCOM），记录设备通信终端信息。通过 UcomInfoDAOImpl/UcomInfoMapper 操作。设备心跳上报时更新。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `ucom_info`  (
                              `ucomid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '上位机帐号',
                              `ip` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                              `cabin_temp` double NULL DEFAULT NULL COMMENT 'cabin温度',
                              `cpu_temp` double NULL DEFAULT NULL COMMENT 'cpu温度',
                              `tx_speed` int(11) NULL DEFAULT NULL COMMENT '上传速度kb/s',
                              `rx_speed` int(11) NULL DEFAULT NULL COMMENT '下载速度kb/s',
                              `ram` int(11) NULL DEFAULT NULL COMMENT '内存kb',
                              `available_ram` int(11) NULL DEFAULT NULL COMMENT '可用内存kb',
                              `hard_disk_space` int(11) NULL DEFAULT NULL COMMENT '硬盘gb',
                              `available_hard_disk_space` int(11) NULL DEFAULT NULL COMMENT '可用硬盘gb',
                              `createtime` bigint(20) NULL DEFAULT NULL COMMENT '创建时间',
                              `updatetime` bigint(20) NULL DEFAULT NULL COMMENT '修改时间',
                              `status` int(2) NULL DEFAULT NULL COMMENT '状态',
                              `case_floor` varchar(28) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '机柜层数',
                              `cpu` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'cpu',
                              `case_num` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '机柜信息'
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '上位机信息' ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

通用通信模块信息表（UCOM），记录设备通信终端信息。通过 UcomInfoDAOImpl/UcomInfoMapper 操作。设备心跳上报时更新。
