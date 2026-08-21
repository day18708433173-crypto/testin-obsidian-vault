---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# view_role_function

- 数据库：`db_service`
- 对象类型：视图

## 用途

角色-功能联合视图。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `service_function`  (
                                     `id` int(11) NOT NULL,
                                     `module_id` int(11) NOT NULL COMMENT '模块ID',
                                     `parent_id` int(11) NOT NULL COMMENT '父节点编号，0-表示目录根节点',
                                     `name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '菜单名称',
                                     `display_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '显示菜单名称，页面优先显示此菜单',
                                     `href` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '链接地址',
                                     `target` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '是否新窗口打开',
                                     `position` int(4) NULL DEFAULT NULL,
                                     `icon` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '菜单项对应的图标',
                                     `status` smallint(2) NULL DEFAULT NULL,
                                     `createtime` bigint(20) NULL DEFAULT NULL,
                                     `updatetime` bigint(20) NULL DEFAULT NULL,
                                     `config_key` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '配置资源key，供页面渲染识别',
                                     `parent_config_key` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '父节点的configkey',
                                     PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Table structure for service_module
-- ----------------------------
DROP TABLE IF EXISTS `service_module`;
CREATE TABLE `service_module`  (
                                   `id` int(11) NOT NULL,
                                   `name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '模块名称',
                                   `domain` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '域名',
                                   `descr` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '描述',
                                   `status` smallint(2) NOT NULL COMMENT '数据状态',
                                   `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                   `updatetime` bigint(20) NOT NULL COMMENT '更新时间',
                                   PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Table structure for service_role
-- ----------------------------
DROP TABLE IF EXISTS `service_role`;
CREATE TABLE `service_role`  (
                                 `id` int(11) NOT NULL,
                                 `name` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                 `status` smallint(2) NOT NULL,
                                 `createtime` bigint(20) NOT NULL,
                                 `updatetime` bigint(20) NOT NULL,
                                 PRIMARY KEY (`id`) USING BTREE,
                                 UNIQUE INDEX `name`(`name`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Table structure for service_role_function
-- ----------------------------
DROP TABLE IF EXISTS `service_role_function`;
CREATE TABLE `service_role_function`  (
                                          `id` int(11) NOT NULL AUTO_INCREMENT,
                                          `fun_id` int(11) NOT NULL,
                                          `role_id` int(11) NOT NULL,
                                          `display_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '显示菜单名称，页面优先显示此菜单',
                                          `hidden` smallint(2) NOT NULL DEFAULT 1 COMMENT '0为显示，1为隐藏（前台页面是否可以查看）',
                                          `status` int(11) NOT NULL,
                                          `createtime` bigint(20) NOT NULL,
                                          `updatetime` bigint(20) NOT NULL,
                                          PRIMARY KEY (`id`) USING BTREE,
                                          UNIQUE INDEX `role_fun`(`role_id`, `fun_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Table structure for service_user_role
-- ----------------------------
DROP TABLE IF EXISTS `service_user_role`;
CREATE TABLE `service_user_role`  (
                                      `id` int(11) NOT NULL,
                                      `eid` int(11) NOT NULL COMMENT '企业ID 0 所有的企业默认使用的配置',
                                      `user_role_id` int(11) NOT NULL COMMENT 'db_user 库中对应的用户角色信息。',
                                      `role_id` int(11) NOT NULL COMMENT 'db_service 库中配置的角色信息',
                                      `status` smallint(2) NOT NULL,
                                      `createtime` bigint(20) NOT NULL,
                                      `updatetime` bigint(20) NOT NULL,
                                      PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- View structure for view_role_function
-- ----------------------------
DROP VIEW IF EXISTS `view_role_function`;
CREATE VIEW `view_role_function` AS select `service_function`.`id` AS `fun_id`,`service_user_role`.`eid` AS `eid`,`service_function`.`module_id` AS `module_id`,`service_module`.`name` AS `module_name`,`service_module`.`domain` AS `domain`,`service_role_function`.`role_id` AS `role_id`,`service_function`.`parent_id` AS `parent_id`,`service_function`.`name` AS `fun_name`,`service_function`.`display_name` AS `display_name`,`service_role_function`.`display_name` AS `role_display_name`,`service_function`.`href` AS `href`,`service_function`.`target` AS `target`,`service_function`.`position` AS `position`,`service_function`.`icon` AS `icon`,`service_role_function`.`hidden` AS `hidden`,`service_function`.`status` AS `status`,`service_function`.`createtime` AS `createtime`,`service_function`.`updatetime` AS `updatetime` from (((`service_function` join `service_role_function` on((`service_function`.`id` = `service_role_function`.`fun_id`))) join `service_module` on((`service_module`.`id` = `service_function`.`module_id`))) left join `service_user_role` on((`service_role_function`.`role_id` = `service_user_role`.`role_id`)));
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`
- `PRIMARY KEY (`id`) USING BTREE`
- `PRIMARY KEY (`id`) USING BTREE`
- `PRIMARY KEY (`id`) USING BTREE`
- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `ViewRoleFunctionDAOImpl.list`（JDBC）← `ScfgServiceImpl` ← 接口 [ServiceCfg](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/ServiceCfg.md).list
