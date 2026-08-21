---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_function_switch

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

功能开关表。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_function_switch`  (
                                            `eid` int(11) NOT NULL DEFAULT 0 COMMENT '企业id',
                                            `config` text CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '企业对应的功能点开关配置',
                                            `descr` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '企业功能开关描述',
                                            `status` int(11) NOT NULL DEFAULT 1 COMMENT '数据状态，0=无效，1=有效',
                                            `expiretime` bigint(20) NOT NULL DEFAULT 0 COMMENT '过期时间，过期的开关全部按照关闭处理',
                                            `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                            `updatetime` bigint(20) NOT NULL DEFAULT 0 COMMENT '最后修改时间',
                                            PRIMARY KEY (`eid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`eid`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgFunctionSwitchDAOImpl`（JDBC）← `FunctionSwitchServiceImpl` ← 接口 [FunctionSwitch](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/FunctionSwitch.md)
