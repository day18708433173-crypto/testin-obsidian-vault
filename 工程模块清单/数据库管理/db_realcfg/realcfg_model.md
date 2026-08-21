---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_model

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

机型库表：平台机型主数据（分辨率、CPU/GPU、传感器能力等）。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_model`  (
                                  `modelid` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键自增，机型ID',
                                  `name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '机型名称',
                                  `alias_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '别名',
                                  `brand_id` int(11) NOT NULL COMMENT '品牌id',
                                  `type` int(11) NOT NULL COMMENT '机型分类信息,phone = 0, pad = 1, tv = 2, other = 9',
                                  `os_name` int(11) NOT NULL COMMENT '系统名称,android = 1, iOS = 2',
                                  `dpi_width` int(11) NOT NULL COMMENT '分辨率宽度',
                                  `dpi_height` int(11) NOT NULL COMMENT '分辨率高度',
                                  `screen_size` double(20, 2) NULL DEFAULT 0.00 COMMENT '屏幕大小',
                                  `pic_url` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '机型图片（小图）',
                                  `pic_url_m` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '机型图片（中图）',
                                  `pic_url_b` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '机型图片（大图）',
                                  `cpu_freq` int(11) NULL DEFAULT NULL COMMENT 'cpu最大频率',
                                  `cpu_num` int(11) NULL DEFAULT NULL COMMENT 'cpu 数量',
                                  `cpu_model` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'cpu 型号',
                                  `cpu_brand` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'cpu 芯片品牌',
                                  `cpu_processor` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'cpu 架构',
                                  `gpu_vendor` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'gpu 厂商',
                                  `gpu_renderer` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'gpu 渲染器',
                                  `gpu_version` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'gpu 版本',
                                  `ram` bigint(20) NULL DEFAULT NULL COMMENT '内存',
                                  `rom` bigint(20) NULL DEFAULT NULL COMMENT 'rom',
                                  `nfc` int(11) NULL DEFAULT NULL COMMENT 'NFC支持,No = 0, Yes = 1',
                                  `bluetooth` int(11) NULL DEFAULT NULL COMMENT '蓝牙支持,No = 0, Yes = 1',
                                  `bluetooth_version` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '蓝牙版本',
                                  `on_shelve_time` bigint(20) NULL DEFAULT NULL COMMENT '机型上架时间',
                                  `off_shelve_time` bigint(20) NULL DEFAULT NULL COMMENT '机型下架时间',
                                  `fingermark` int(11) NULL DEFAULT 0 COMMENT '指纹识别,No = 0, Yes = 1',
                                  `otg` int(11) NULL DEFAULT 0 COMMENT 'OTG,No = 0, Yes = 1',
                                  `logical_resolution` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                  `status` int(11) NOT NULL COMMENT '状态,off = 0, on = 1',
                                  `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                  `updatetime` bigint(20) NOT NULL COMMENT '更新时间',
                                  `weight` int(11) NOT NULL DEFAULT 1 COMMENT '机型权重信息，用于查询列表排序',
                                  PRIMARY KEY (`modelid`) USING BTREE,
                                  UNIQUE INDEX `ind_realcfg_model`(`name`, `brand_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`modelid`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgModelDAOImpl`（JDBC）← `ModelServiceImpl` ← 接口 [Model](../../平台配置（real-cfg）/07-开放接口文档/项目与平台配置/Model.md)
- 被视图 [view_model_brand](view_model_brand.md) 引用
