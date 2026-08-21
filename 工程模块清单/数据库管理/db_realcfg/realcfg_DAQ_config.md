---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_DAQ_config

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

DAQ 采集配置表：数据采集通道配置。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_DAQ_config`  (
                                       `id` int(11) NOT NULL AUTO_INCREMENT COMMENT 'DAQ 主键',
                                       `eid` int(11) NULL DEFAULT NULL COMMENT '企业id',
                                       `project_id` int(11) NULL DEFAULT NULL COMMENT '项目组id',
                                       `semantics` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '语义',
                                       `creator` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '创建人',
                                       `creator_id` int(11) NULL DEFAULT NULL COMMENT '创建人id',
                                       `descr` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '备注',
                                       `createtime` bigint(20) NOT NULL,
                                       `updatetime` bigint(20) NULL DEFAULT NULL,
                                       PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgDAQConfigDAOImpl`（JDBC）← `RealCfgDAQConfigServiceImpl` ← 接口 [DAQCfg](../../平台配置（real-cfg）/07-开放接口文档/数据源与代码配置/DAQCfg.md)
