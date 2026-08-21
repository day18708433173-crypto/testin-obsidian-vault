---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_coding

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

设备编码配置表。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_coding`  (
                                   `code` int(11) NOT NULL COMMENT '执测单元编码',
                                   `pf_code` int(11) NOT NULL COMMENT '平台对应编码',
                                   `result_category` int(11) NOT NULL COMMENT '结果分类',
                                   `status` int(11) NOT NULL COMMENT '数据状态,off = 0, on = 1',
                                   `descr` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '描述',
                                   `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                   `updatetime` bigint(2) NOT NULL COMMENT '更新时间',
                                   PRIMARY KEY (`code`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`code`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgCodingDAOImpl`（JDBC）← `CodingServiceImpl` ← 接口 [CodingCfg](../../平台配置（real-cfg）/07-开放接口文档/数据源与代码配置/CodingCfg.md)
