---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_sourcecode_config

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

源码配置表。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_sourcecode_config`  (
                                              `id` int(11) NOT NULL AUTO_INCREMENT,
                                              `name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '源码项目名称',
                                              `descr` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '源码项目描述',
                                              `eid` int(11) NOT NULL COMMENT '用户企业id',
                                              `project_id` int(11) NULL DEFAULT NULL COMMENT '项目组ID 0 为企业级别信息。',
                                              `project_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '用户项目组名称',
                                              `vcs_type` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '版本控制分类-git',
                                              `repository_url` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '仓库地址',
                                              `branch_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '分支名称',
                                              `authentication_type` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '鉴权类型-Anonymous、Password、SSHKey',
                                              `account_id` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '账户ID',
                                              `pwd` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '账户密码',
                                              `ssh_key` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
                                              `opr_userid` int(11) NOT NULL COMMENT '操作人id',
                                              `opr_username` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '操作人名称',
                                              `status` int(11) NOT NULL,
                                              `createtime` bigint(20) NOT NULL,
                                              `updatetime` bigint(20) NOT NULL,
                                              PRIMARY KEY (`id`) USING BTREE,
                                              UNIQUE INDEX `eid_name`(`eid`, `name`) USING BTREE,
                                              INDEX `eid_pid`(`eid`, `project_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 2 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgSourcecodeConfigDAOImpl`（JDBC）← `SourcecodeConfigServiceImpl` ← 接口 [SourcecodeConfig](SourcecodeConfig.md)
