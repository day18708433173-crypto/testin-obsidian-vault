---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# client_info_refresh

## 用途

客户端信息刷新辅助表。

## 所属数据库

db_client

## DDL

```sql
CREATE TABLE `client_info_refresh`  (
                                        `pool_id` int(11) NOT NULL,
                                        `vhost` int(11) NOT NULL,
                                        `ucomid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '' COMMENT '上位机账号',
                                        `ip` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                        `location` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '位置信息',
                                        `system_type` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '系统类型',
                                        `system_version` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '系统版本',
                                        `system_name` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '系统名称',
                                        `pc_id` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '设备id',
                                        `system_bitness` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '系统位数',
                                        `cpu_name` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'cpu名称',
                                        `cpu_arch` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'cpu架构',
                                        `ram` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '内存',
                                        `brand_name` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '电脑品牌',
                                        `action` int(11) NULL DEFAULT NULL COMMENT '动作： 0 空闲; 1 测试; 2真机调试 3 online 4 第三方',
                                        `debug_owner` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '占用者信息：\n真机、online等为用户email信息\n执行任务为对应的子任务信息',
                                        `licences` int(11) NULL DEFAULT NULL,
                                        `source_rule` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                        `status` int(11) NULL DEFAULT NULL,
                                        `createtime` bigint(20) NULL DEFAULT NULL,
                                        `updatetime` bigint(20) NULL DEFAULT NULL,
                                        `refreshtime` bigint(20) NULL DEFAULT NULL,
                                        `protocol` varchar(20) NULL COMMENT '上位机连接方式',
                                        `marks` varchar(2000) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                        PRIMARY KEY (`pool_id`, `ucomid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_client.sql（命中）

## 设备控制中心 中的使用

客户端信息刷新辅助表。
