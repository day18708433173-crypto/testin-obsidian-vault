---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_device_model

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

企业设备机型表：企业自有设备机型登记。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_device_model`  (
                                         `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键，自增id，',
                                         `modelid` int(11) NULL DEFAULT NULL COMMENT '机型id',
                                         `series_name` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '系列名',
                                         `model_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '机型名称',
                                         `dpi_width` int(11) NULL DEFAULT NULL COMMENT '分辨率宽度',
                                         `dpi_height` int(11) NULL DEFAULT NULL COMMENT '分辨率高度',
                                         `screen_size` double(20, 2) NULL DEFAULT NULL COMMENT '屏幕大小',
                                         `cpu` int(11) NULL DEFAULT NULL COMMENT 'cpu',
                                         `cpu_num` int(11) NULL DEFAULT NULL COMMENT 'cpu数量',
                                         `ram` bigint(20) NULL DEFAULT NULL COMMENT '内存',
                                         `status` int(11) NOT NULL COMMENT '状态,off = 0, on = 1',
                                         `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                         `updatetime` bigint(20) NOT NULL COMMENT '更新时间',
                                         PRIMARY KEY (`id`) USING BTREE,
                                         UNIQUE INDEX `sname_mname`(`series_name`, `model_name`) USING BTREE,
                                         INDEX `modelid`(`modelid`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgDeviceModelDAOImpl`（JDBC）← `DeviceModelServiceImpl` ← 接口 [DeviceModel](../../平台配置（real-cfg）/07-开放接口文档/设备与网络配置/DeviceModel.md)
