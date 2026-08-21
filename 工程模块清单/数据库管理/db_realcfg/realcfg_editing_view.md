---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_editing_view

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

编辑视图配置表。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_editing_view`  (
                                         `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '自增id',
                                         `module_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '页面模块名称',
                                         `key` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '采编页面显示的key',
                                         `value` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '采编页面显示的内容',
                                         `show_sequence` int(11) NOT NULL DEFAULT 1 COMMENT '模块下的显示顺序',
                                         `expire_time` bigint(20) NOT NULL DEFAULT 0 COMMENT '过期时间，0=永久',
                                         `status` int(11) NOT NULL DEFAULT 1 COMMENT '数据状态，0=无效，1=有效',
                                         `createtime` bigint(20) NULL DEFAULT NULL COMMENT '数据创建时间',
                                         `updatetime` bigint(20) NULL DEFAULT NULL COMMENT '数据更新时间',
                                         PRIMARY KEY (`id`) USING BTREE,
                                         UNIQUE INDEX `key`(`key`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgEditingViewDAOImpl`（JDBC）← `EditingViewServiceImpl` ← 接口 [EditingView](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/EditingView.md)
