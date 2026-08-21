---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_brand

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

设备品牌表：品牌名称、缩写、logo、拼音。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_brand`  (
                                  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键自增,品牌id',
                                  `name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '品牌名称',
                                  `abbr` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '品牌别名',
                                  `spelling` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '品牌拼音',
                                  `logo_url` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT 'logoUrl信息',
                                  `weight` int(11) NULL DEFAULT 0 COMMENT '权重信息',
                                  `status` int(11) NOT NULL COMMENT '状态,off = 0, on = 1',
                                  `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                  `updatetime` bigint(20) NOT NULL COMMENT '更新时间',
                                  PRIMARY KEY (`id`) USING BTREE,
                                  UNIQUE INDEX `name`(`name`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgBrandDAOImpl`（JDBC）← `BrandServiceImpl` ← 接口 [Brand](../../平台配置（real-cfg）/07-开放接口文档/项目与平台配置/Brand.md)
- 被视图 [view_model_brand](view_model_brand.md) 引用
