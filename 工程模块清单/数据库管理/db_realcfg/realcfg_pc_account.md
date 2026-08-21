---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_pc_account

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

真机节点（ucomid）账号表：节点签到/签退状态。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_pc_account`  (
                                       `ucomid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '上位机账号',
                                       `ucomid_pwd` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '上位机密码',
                                       `descr` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '描述',
                                       `sign` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                                       `signvhost` int(11) NULL DEFAULT NULL,
                                       `signtime` bigint(20) NULL DEFAULT NULL,
                                       `signouttime` bigint(20) NULL DEFAULT NULL,
                                       `status` int(11) NOT NULL COMMENT '状态,off = 0, on = 1',
                                       `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                       `updatetime` bigint(20) NOT NULL COMMENT '更新时间',
                                       `lastaccesstime` bigint(20) NULL DEFAULT NULL,
                                       PRIMARY KEY (`ucomid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`ucomid`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgPcAccountDAOImpl`（JDBC）← `PcAccountServiceImpl` ← 接口 [PcAccount](../../平台配置（real-cfg）/07-开放接口文档/用户与权限/PcAccount.md)
- 被视图 [view_pc_account](view_pc_account.md) 引用
