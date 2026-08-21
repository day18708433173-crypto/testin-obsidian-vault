---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# mcfg_role

- 数据库：`db_mcfg`
- 对象类型：表

## 用途

openapi 角色表。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `mcfg_role`  (
                              `id` int(11) NOT NULL AUTO_INCREMENT,
                              `name` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                              `status` tinyint(4) NOT NULL,
                              `createtime` bigint(20) NOT NULL,
                              `updatetime` bigint(20) NOT NULL,
                              PRIMARY KEY (`id`) USING BTREE,
                              UNIQUE INDEX `name`(`name`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `McfgRoleDAOImpl`（JDBC）← `RoleServiceImpl` ← 接口 [RoleCfg](../../平台配置（real-cfg）/07-开放接口文档/用户与权限/RoleCfg.md)（add/delete/maintain/get/list、addApi 校验）
