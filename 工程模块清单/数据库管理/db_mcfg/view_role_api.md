---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# view_role_api

- 数据库：`db_mcfg`
- 对象类型：视图

## 用途

角色-接口联合视图。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `mcfg_api`  (
                             `id` int(11) NOT NULL,
                             `module_id` int(11) NOT NULL,
                             `api_action` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                             `api_op` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                             `special_api_action` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                             `special_api_op` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                             `descr` varchar(254) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
                             `need_login` tinyint(4) NOT NULL DEFAULT 1 COMMENT '是否需要登录使用',
                             `need_activation` tinyint(4) NOT NULL DEFAULT 1 COMMENT '否是激活才能使用',
                             `need_write` tinyint(4) NOT NULL COMMENT '是否有写库的操作',
                             `protocol_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '赋值配置：\"additionalKeys\":{\"name\":[\"key1\",\"key2\",\"key3\",\"key4\"],\"objInfo\":{\"name\":\"qianggao\"}},\"ignoreKeys|fieldKeys|checkKeys\":{\"info\":{\"name\",\"obj\":{\"k1\",\"k2\"}}}',
                             `rule_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '暂不启用',
                             `access_frequency` int(11) NOT NULL DEFAULT 0 COMMENT '0 默认不限; 执行频率按照访问的人',
                             `access_quota` int(11) NOT NULL,
                             `quota_unit` bigint(20) NOT NULL,
                             `status` tinyint(4) NOT NULL,
                             `createtime` bigint(20) NOT NULL,
                             `updatetime` bigint(20) NOT NULL,
                             `pass_through_type` int(11) NOT NULL DEFAULT '0',
                             PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Table structure for mcfg_app
-- ----------------------------
DROP TABLE IF EXISTS `mcfg_app`;
CREATE TABLE `mcfg_app`  (
                             `id` int(11) NOT NULL AUTO_INCREMENT,
                             `app_name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
                             `api_key` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
                             `app_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT 'app相关配置',
                             `secret_key` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
                             `iv_key` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
                             `ips` text CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                             `bind_eid` int(11) NOT NULL,
                             `roles` text CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '角色配置[1,3,4]',
                             `description` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT '',
                             `status` tinyint(4) NOT NULL DEFAULT 0,
                             `createtime` bigint(20) NOT NULL,
                             `updatetime` bigint(20) NOT NULL,
                             PRIMARY KEY (`id`) USING BTREE,
                             UNIQUE INDEX `apikey`(`api_key`) USING BTREE,
                             UNIQUE INDEX `appname`(`app_name`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Table structure for mcfg_module
-- ----------------------------
DROP TABLE IF EXISTS `mcfg_module`;
CREATE TABLE `mcfg_module`  (
                                `id` int(11) NOT NULL,
                                `mkey` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                `name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                `http_hosts` varchar(2048) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `rpc_prefix_name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                `distributed` int(11) NOT NULL COMMENT '是否是数据分布部署',
                                `rule_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '数据分布部署的规则',
                                `descr` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                `status` int(11) NOT NULL,
                                `createtime` bigint(20) NOT NULL,
                                `updatetime` bigint(20) NOT NULL,
                                PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Table structure for mcfg_role
-- ----------------------------
DROP TABLE IF EXISTS `mcfg_role`;
CREATE TABLE `mcfg_role`  (
                              `id` int(11) NOT NULL AUTO_INCREMENT,
                              `name` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                              `status` tinyint(4) NOT NULL,
                              `createtime` bigint(20) NOT NULL,
                              `updatetime` bigint(20) NOT NULL,
                              PRIMARY KEY (`id`) USING BTREE,
                              UNIQUE INDEX `name`(`name`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Table structure for mcfg_role_api
-- ----------------------------
DROP TABLE IF EXISTS `mcfg_role_api`;
CREATE TABLE `mcfg_role_api`  (
                                  `id` int(11) NOT NULL AUTO_INCREMENT,
                                  `role_id` int(11) NOT NULL,
                                  `api_id` int(11) NOT NULL,
                                  `protocol_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '见mcfg_api表中protocol_config',
                                  `access_frequency` int(11) NOT NULL DEFAULT 0,
                                  `access_quota` int(11) NOT NULL,
                                  `quota_unit` bigint(20) NOT NULL COMMENT '配额时间单位',
                                  `status` tinyint(4) NOT NULL,
                                  `createtime` bigint(20) NOT NULL,
                                  `updatetime` bigint(20) NOT NULL,
                                  PRIMARY KEY (`id`) USING BTREE,
                                  UNIQUE INDEX `role_api`(`role_id`, `api_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- View structure for view_role_api
-- ----------------------------
DROP VIEW IF EXISTS `view_role_api`;
CREATE OR REPLACE  VIEW `view_role_api` AS select `mcfg_role_api`.`role_id` AS `role_id`,`mcfg_role_api`.`api_id` AS `api_id`,`mcfg_api`.`module_id` AS `module_id`,`mcfg_api`.`api_action` AS `api_action`,`mcfg_api`.`api_op` AS `api_op`,`mcfg_api`.`need_login` AS `need_login`,`mcfg_api`.`need_activation` AS `need_activation`,`mcfg_api`.`need_write` AS `need_write`,`mcfg_api`.`protocol_config` AS `protocol_config`,`mcfg_api`.`rule_config` AS `rule_config`,`mcfg_api`.`access_frequency` AS `access_frequency`,`mcfg_api`.`access_quota` AS `access_quota`,`mcfg_api`.`quota_unit` AS `quota_unit`,`mcfg_role_api`.`protocol_config` AS `role_protocol_config`,`mcfg_role_api`.`access_frequency` AS `role_access_frequency`,`mcfg_role_api`.`access_quota` AS `role_access_quota`,`mcfg_role_api`.`quota_unit` AS `role_quota_unit`,`mcfg_api`.`descr` AS `descr`,`mcfg_api`.`status` AS `status`,`mcfg_api`.`createtime` AS `createtime`,`mcfg_api`.`updatetime` AS `updatetime`,`mcfg_api`.`special_api_action` AS `special_api_action`,`mcfg_api`.`special_api_op` AS `special_api_op`, `mcfg_api`.`pass_through_type` AS `pass_through_type` from (`mcfg_api` join `mcfg_role_api` on((`mcfg_api`.`id` = `mcfg_role_api`.`api_id`)));
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`
- `PRIMARY KEY (`id`) USING BTREE`
- `PRIMARY KEY (`id`) USING BTREE`
- `PRIMARY KEY (`id`) USING BTREE`
- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `ViewRoleApiDAOImpl`（JDBC）← `ApiServiceImpl` ← 接口 [ApiCfg](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/ApiCfg.md).listByRole
- `RoleServiceImpl.delete` 删除角色前做占用校验（接口 [RoleCfg](../../平台配置（real-cfg）/07-开放接口文档/用户与权限/RoleCfg.md).delete）
