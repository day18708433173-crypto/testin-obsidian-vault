---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# mcfg_role_api

- 数据库：`db_mcfg`
- 对象类型：表

## 用途

openapi 角色-接口授权表：含角色级协议配置 protocolConfig、频控/配额字段。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `mcfg_role_api`  (
                                  `id` int(11) NOT NULL AUTO_INCREMENT,
                                  `role_id` int(11) NOT NULL,
                                  `api_id` int(11) NOT NULL,
                                  `protocol_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '见mcfg_api表中protocol_config',
                                  `access_frequency` int(11) NOT NULL DEFAULT 0,
                                  `access_quota` int(11) NOT NULL,
                                  `quota_unit` bigint(20) NOT NULL COMMENT '配额时间单位',
                                  `status` tinyint(4) NOT NULL,
                                  `createtime` bigint(20) NOT NULL,
                                  `updatetime` bigint(20) NOT NULL,
                                  PRIMARY KEY (`id`) USING BTREE,
                                  UNIQUE INDEX `role_api`(`role_id`, `api_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `McfgRoleApiDAOImpl`（JDBC）← `RoleServiceImpl` ← 接口 [RoleCfg](../../平台配置（real-cfg）/07-开放接口文档/用户与权限/RoleCfg.md)（maintainApi/addApi/removeApi）
