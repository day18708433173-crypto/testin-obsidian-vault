---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_network_simulation

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

弱网/网络模拟配置表。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_network_simulation`  (
                                               `eid` int(11) NOT NULL,
                                               `name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '网络类型名称',
                                               `rate` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '网速',
                                               `delay` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '延迟',
                                               `loss` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '丢包率',
                                               `corruption` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '损坏',
                                               `reorder` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '重排序',
                                               `write` smallint(6) NOT NULL DEFAULT 0 COMMENT '数据是否可写，可写数据在后台可以修改上下行取值，0=不可写，1=可写',
                                               `status` smallint(6) NOT NULL DEFAULT 1 COMMENT '数据状态，0=无效，1=有效',
                                               `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                               `updatetime` bigint(20) NOT NULL COMMENT '最后更新时间',
                                               PRIMARY KEY (`eid`, `name`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`eid`, `name`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `DbRealcfgNetworkSimulationDAOImpl`（JDBC）← `NetworkSimulationServiceImpl` ← 接口 [NetworkCfg](../../平台配置（real-cfg）/07-开放接口文档/设备与网络配置/NetworkCfg.md)
