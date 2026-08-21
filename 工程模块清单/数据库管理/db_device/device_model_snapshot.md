---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_model_snapshot

## 用途

设备机型快照表（品牌、型号、分辨率、CPU、GPU、RAM、ROM、NFC等详细参数）。用于 view_device_info/view_device_source_info 视图 LEFT JOIN 补充机型信息。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_model_snapshot`  (
                                          `modelid` int(11) NOT NULL,
                                          `brand_id` int(11) NOT NULL,
                                          `brand_name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                          `brand_abbr` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                          `logo_url` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
                                          `weight` int(11) NULL DEFAULT NULL,
                                          `model_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                          `model_alias` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                          `model_type` int(11) NOT NULL,
                                          `os_name` int(11) NOT NULL,
                                          `dpi_width` int(11) NULL DEFAULT NULL,
                                          `dpi_height` int(11) NULL DEFAULT NULL,
                                          `screen_size` double NULL DEFAULT NULL,
                                          `pic_url` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                          `pic_url_m` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                          `pic_url_b` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                          `cpu_freq` int(11) NULL DEFAULT NULL,
                                          `cpu_num` int(11) NULL DEFAULT NULL,
                                          `cpu_model` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                          `cpu_brand` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                          `cpu_processor` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                          `gpu_vendor` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                          `gpu_renderer` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'gpu 渲染器',
                                          `gpu_version` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                          `ram` bigint(20) NULL DEFAULT NULL,
                                          `rom` bigint(20) NULL DEFAULT NULL,
                                          `nfc` int(11) NULL DEFAULT NULL,
                                          `bluetooth` int(11) NULL DEFAULT NULL,
                                          `bluetooth_version` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                          `on_shelve_time` bigint(20) NULL DEFAULT NULL,
                                          `off_shelve_time` bigint(20) NULL DEFAULT NULL,
                                          `model_version` bigint(20) NOT NULL,
                                          `fingermark` int(11) NULL DEFAULT 0 COMMENT '指纹识别,No = 0, Yes = 1',
                                          `otg` int(11) NULL DEFAULT 0 COMMENT 'OTG,No = 0, Yes = 1',
                                          `logical_resolution` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                          `status` int(11) NOT NULL,
                                          `createtime` bigint(20) NOT NULL,
                                          `updatetime` bigint(20) NOT NULL,
                                          PRIMARY KEY (`modelid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备机型快照表（品牌、型号、分辨率、CPU、GPU、RAM、ROM、NFC等详细参数）。用于 view_device_info/view_device_source_info 视图 LEFT JOIN 补充机型信息。
