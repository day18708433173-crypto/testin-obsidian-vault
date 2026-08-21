---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL视图
---

# view_pc_info

## 用途

PC信息-浏览器联合视图（pc_info JOIN pc_browser_info）。ViewPcInfoDAOImpl 和 ViewPcInfoMapper 查询使用。

## 所属数据库

db_pc

## DDL

```sql
CREATE TABLE `browser_process`  (
                                    `id` bigint(20) UNSIGNED NOT NULL AUTO_INCREMENT,
                                    `job_id` varchar(128) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '任务id',
                                    `vhost` int(11) NULL DEFAULT NULL COMMENT '设备连接节点编号',
                                    `eid` int(11) NULL DEFAULT NULL COMMENT '企业id',
                                    `projectid` int(11) NULL DEFAULT NULL COMMENT '项目id\n项目id\n',
                                    `ucomid` varchar(128) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '上位机账号',
                                    `ip` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '设备ip',
                                    `device_action` int(11) NULL DEFAULT NULL COMMENT '1真机2online录制脚本3测试4第三方使用\n',
                                    `deviceid` varchar(128) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '设备id\n',
                                    `os_name` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '设备系统名称',
                                    `os_version` varchar(128) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '设备系统版本',
                                    `browser_type` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '浏览器类型\n浏览器类型\n',
                                    `browser_version` varchar(128) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '浏览器版本\n',
                                    `additional_content` text CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL COMMENT '附加信息',
                                    `userid` int(11) NULL DEFAULT NULL COMMENT '用户ID\n',
                                    `user_name` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '用户名\n',
                                    `user_email` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '用户邮箱\n',
                                    `totaltime` bigint(20) NULL DEFAULT NULL COMMENT '连接总时间',
                                    `descr` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '9异常断开，2一场点开0主动断开\n',
                                    `mark_name_1` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '附加信息，默认为null\n',
                                    `mark_name_2` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
                                    `status` int(11) NULL DEFAULT NULL COMMENT '0 执行中 1 已完成',
                                    `createtime` bigint(20) NULL DEFAULT NULL COMMENT '创建时间',
                                    `updatetime` bigint(20) NULL DEFAULT NULL COMMENT '修改时间',
                                    PRIMARY KEY (`id`) USING BTREE,
                                    INDEX `job_id`(`job_id`) USING BTREE,
                                    INDEX `status_deviceid`(`status`, `deviceid`) USING BTREE,
                                    INDEX `status_vhost`(`status`, `vhost`) USING BTREE,
                                    INDEX `updatetime_status`(`updatetime`, `status`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for pc_browser_info
-- ----------------------------
DROP TABLE IF EXISTS `pc_browser_info`;
CREATE TABLE `pc_browser_info`  (
                                    `ucomid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                    `type` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                    `version` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                    `createtime` bigint(20) NOT NULL,
                                    PRIMARY KEY (`ucomid`, `type`, `version`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Table structure for pc_info
-- ----------------------------
DROP TABLE IF EXISTS `pc_info`;
CREATE TABLE `pc_info`  (
                            `vhost` int(11) NOT NULL,
                            `ucomid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '' COMMENT '上位机账号',
                            `ip` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                            `location` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '位置信息',
                            `os_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '系统名称',
                 
-- ... (truncated)
```

> DDL 来源：pocinit/src/mysql/db_pc.sql（命中）

## 设备控制中心 中的使用

PC信息-浏览器联合视图（pc_info JOIN pc_browser_info）。ViewPcInfoDAOImpl 和 ViewPcInfoMapper 查询使用。
