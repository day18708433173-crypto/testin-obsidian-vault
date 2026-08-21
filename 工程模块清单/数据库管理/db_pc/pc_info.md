---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# pc_info

## 用途

PC（上位机）主信息表。PcWorkerThread 维护缓存超时清理，PcDataThread 清理DB过期记录，PcHandlerThread 任务匹配查询。PcAccountWorkerThread 每30秒上报/清理账号状态。通过 AbstractGenericDaoImpl.getPcdao() 访问。

## 所属数据库

db_pc

## DDL

```sql
CREATE TABLE `pc_info`  (
                            `vhost` int(11) NOT NULL,
                            `ucomid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '' COMMENT '上位机账号',
                            `ip` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                            `location` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '位置信息',
                            `os_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '系统名称',
                            `os_version` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '系统版本',
                            `action` int(11) NOT NULL COMMENT '动作： 0 空闲; 1 测试; 2真机调试 3 online 4 第三方',
                            `debug_owner` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '占用者信息：\n真机、online等为用户email信息\n执行任务为对应的子任务信息',
                            `browsers` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '使用中的浏览器信息:[{},{}]\n',
                            `licences` int(11) NOT NULL,
                            `source_rule` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                            `status` int(11) NOT NULL,
                            `createtime` bigint(20) NOT NULL,
                            `updatetime` bigint(20) NOT NULL,
                            `refreshtime` bigint(20) NOT NULL,
                            `protocol` varchar(20) NULL COMMENT '上位机连接方式',
                            `marks` varchar(2000) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                            PRIMARY KEY (`ucomid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_pc.sql（命中）

## 设备控制中心 中的使用

PC（上位机）主信息表。PcWorkerThread 维护缓存超时清理，PcDataThread 清理DB过期记录，PcHandlerThread 任务匹配查询。PcAccountWorkerThread 每30秒上报/清理账号状态。通过 AbstractGenericDaoImpl.getPcdao() 访问。
