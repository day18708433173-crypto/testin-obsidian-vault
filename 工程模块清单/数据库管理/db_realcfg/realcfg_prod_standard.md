---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_prod_standard

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

产品标准配置表。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_prod_standard`  (
                                          `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '自增主键',
                                          `name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '名称',
                                          `status` int(11) NOT NULL COMMENT '数据状态',
                                          `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                          `updatetime` bigint(20) NOT NULL COMMENT '更新时间',
                                          PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- 本模块仅有 pojo `RealcfgProdStandard`，未见 DAO/接口直接使用（推断供其他模块或历史功能使用）
