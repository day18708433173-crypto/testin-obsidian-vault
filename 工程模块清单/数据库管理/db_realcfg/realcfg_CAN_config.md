---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_CAN_config

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

CAN 信号配置表：企业/项目级 CAN 报文语义-信号映射及默认值、取值范围。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_CAN_config`  (
                                       `id` int(11) NOT NULL AUTO_INCREMENT COMMENT 'can 主键',
                                       `eid` int(11) NULL DEFAULT NULL COMMENT '企业id',
                                       `project_id` int(11) NULL DEFAULT NULL COMMENT '项目组id',
                                       `semantics` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '语义',
                                       `signal_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '信号名',
                                       `default_value` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '默认值',
                                       `value_range` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '取值范围',
                                       `creator` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '创建人',
                                       `creator_id` int(11) NULL DEFAULT NULL COMMENT '创建人id',
                                       `descr` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '备注',
                                       `status` tinyint(4) NOT NULL DEFAULT 1,
                                       `createtime` bigint(20) NOT NULL,
                                       `updatetime` bigint(20) NULL DEFAULT NULL,
                                       `type` int(1) NULL DEFAULT NULL COMMENT '// 渠道 1：PW 创建而来  ---  null : CANCfg 创建而来',
                                       PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgCANConfigDAOImpl`（JDBC）← `RealCfgCANConfigServiceImpl` ← 接口 [CANCfg](../../平台配置（real-cfg）/07-开放接口文档/数据源与代码配置/CANCfg.md)
