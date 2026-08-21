---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_timeout_config

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

超时配置表：各类操作超时阈值配置。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_timeout_config`  (
                                                      `id` bigint(20) NOT NULL AUTO_INCREMENT,
                                                      `eid` int(20) DEFAULT NULL COMMENT '企业id',
                                                      `project_id` int(11) DEFAULT NULL COMMENT '项目组id',
                                                      `name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL COMMENT '配置名称',
                                                      `business_type` int(11) DEFAULT NULL COMMENT '1APP 2WEB 3PC',
                                                      `value` int(11) DEFAULT NULL COMMENT '超时时间数值',
                                                      `unit` int(11) DEFAULT NULL COMMENT '单位 1秒,2分,3小时,4天',
                                                      `status` tinyint(4) NOT NULL DEFAULT 1 COMMENT '0：逻辑删除  1：正常',
                                                      `createtime` bigint(20) NOT NULL,
                                                      `updatetime` bigint(20) NOT NULL,
                                                      PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '超时时间配置表' ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgTimeoutConfigDAOImpl`（JDBC）← `RealCfgTimeoutConfigServiceImpl` ← 接口 [Timeout](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/Timeout.md)
