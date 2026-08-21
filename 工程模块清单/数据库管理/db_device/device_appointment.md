---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_appointment

## 用途

设备预约表。AppointmentWorkerThread 查询异常预约并处理、查询待处理预约列表。AppointmentConfigInitThread 加载预约配置。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `device_appointment`  (
                                       `id` int(11) NOT NULL AUTO_INCREMENT,
                                       `eid` int(11) NULL DEFAULT NULL,
                                       `projectid` int(11) NULL DEFAULT NULL,
                                       `type` int(11) NULL DEFAULT NULL COMMENT '设备端分类：（1 app; 2 web；3 pc）',
                                       `ucomid` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                       `deviceid` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                       `device_info` varchar(2000) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '设备信息',
                                       `start_time` bigint(20) NULL DEFAULT NULL,
                                       `end_time` bigint(20) NULL DEFAULT NULL,
                                       `descr` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '预约描述',
                                       `applicant_userid` int(11) NULL DEFAULT NULL COMMENT '申请人userid',
                                       `applicant_email` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '申请人邮箱',
                                       `applicant_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '申请人名称',
                                       `exec_status` int(11) NULL DEFAULT NULL COMMENT '执行状态：-2 异常数据;  -1 审批未通过; 0 申请;  1 审批通过;   2 正在使用;  3 正常结束;  4 过期未使用系统取消; 5 使用过程中强制中断（系统管理员）',
                                       `expire_time` bigint(20) NULL DEFAULT NULL COMMENT '预约过期保留设备时间',
                                       `notice_status` int(255) NULL DEFAULT 0 COMMENT '预约提醒状态：0 未通知；1 已通知\n\n',
                                       `notice_time` bigint(20) NULL DEFAULT NULL COMMENT '预约提醒时间\n',
                                       `operator_userid` int(11) NULL DEFAULT NULL COMMENT '操作员 userid',
                                       `operator_email` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '操作员邮箱',
                                       `operator_descr` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '操作描述',
                                       `req_id` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '数据标识（请求id续约时可使用同一个）',
                                       `handle_content` varchar(1000) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '处理内容信息：(k, v)',
                                       `status` int(11) NULL DEFAULT NULL,
                                       `createtime` bigint(20) NULL DEFAULT NULL,
                                       `updatetime` bigint(20) NULL DEFAULT NULL,
                                       PRIMARY KEY (`id`) USING BTREE,
                                       UNIQUE INDEX `req_id`(`req_id`) USING BTREE,
                                       INDEX `time_zone`(`start_time`, `end_time`, `deviceid`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备预约表。AppointmentWorkerThread 查询异常预约并处理、查询待处理预约列表。AppointmentConfigInitThread 加载预约配置。
