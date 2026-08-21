---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL视图
---

# view_device_info

## 用途

设备-机型联合视图（device_info LEFT JOIN device_model_snapshot LEFT JOIN device_assets_info）。ViewDeviceInfoDAOImpl 查询使用。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `cabinet_space_cfg`  (
                                      `case_floor` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '机柜层数',
                                      `case_row` int(11) NULL DEFAULT NULL COMMENT '行数',
                                      `case_col` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '列数',
                                      `case_num` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '机柜信息'
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '机柜空间配置信息' ROW_FORMAT = Compact;

-- ----------------------------
-- Table structure for certificate_info
-- ----------------------------
DROP TABLE IF EXISTS `certificate_info`;
CREATE TABLE `certificate_info`  (
                                     `certificate_id` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '证书的id',
                                     `name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '证书名称',
                                     `type` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '类型',
                                     `appid` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '应用id',
                                     `package_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '包名',
                                     `certificate_num` int(11) NULL DEFAULT NULL COMMENT '证书数量',
                                     `descr` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '描述',
                                     `certificate_identifier` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '证书标识符',
                                     `expiretime` bigint(20) NULL DEFAULT NULL COMMENT '到期时间',
                                     `fileid` bigint(20) NULL DEFAULT NULL COMMENT '文件id',
                                     `file_md5` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '文件md5',
                                     `file_url` varchar(300) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '文件地址',
                                     `status` int(11) NOT NULL COMMENT '状态，0无效，1有效',
                                     `createtime` bigint(20) NOT NULL,
                                     `updatetime` bigint(20) NOT NULL,
                                     PRIMARY KEY (`certificate_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Table structure for device_appointment
-- ----------------------------
DROP TABLE IF EXISTS `device_appointment`;
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
                                       `operator_email` varchar(25
-- ... (truncated)
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

设备-机型联合视图（device_info LEFT JOIN device_model_snapshot LEFT JOIN device_assets_info）。ViewDeviceInfoDAOImpl 查询使用。
