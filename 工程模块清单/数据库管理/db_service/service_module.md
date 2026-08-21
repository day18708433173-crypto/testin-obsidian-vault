---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# service_module

- 数据库：`db_service`
- 对象类型：表

## 用途

服务模块表：功能所属业务模块及域名。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `service_module`  (
                                   `id` int(11) NOT NULL,
                                   `name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '模块名称',
                                   `domain` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '域名',
                                   `descr` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '描述',
                                   `status` smallint(2) NOT NULL COMMENT '数据状态',
                                   `createtime` bigint(20) NOT NULL COMMENT '创建时间',
                                   `updatetime` bigint(20) NOT NULL COMMENT '更新时间',
                                   PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `ViewRoleFunctionDAOImpl.listOfcompatible` 中 LEFT JOIN（接口 [ServiceCfg](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/ServiceCfg.md).list）
- 被视图 [view_role_function](view_role_function.md) 引用
