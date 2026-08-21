---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# service_role_function

- 数据库：`db_service`
- 对象类型：表

## 用途

角色-功能授权表。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `service_role_function`  (
                                          `id` int(11) NOT NULL AUTO_INCREMENT,
                                          `fun_id` int(11) NOT NULL,
                                          `role_id` int(11) NOT NULL,
                                          `display_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '显示菜单名称，页面优先显示此菜单',
                                          `hidden` smallint(2) NOT NULL DEFAULT 1 COMMENT '0为显示，1为隐藏（前台页面是否可以查看）',
                                          `status` int(11) NOT NULL,
                                          `createtime` bigint(20) NOT NULL,
                                          `updatetime` bigint(20) NOT NULL,
                                          PRIMARY KEY (`id`) USING BTREE,
                                          UNIQUE INDEX `role_fun`(`role_id`, `fun_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RoleFunctionDAOImpl`（JDBC）← `ScfgServiceImpl` ← 接口 [ServiceCfg](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/ServiceCfg.md)（listRoleFunction/insertRoleFunction/listRoleControl/openCloseFunction）
- `ViewRoleFunctionDAOImpl.listOfcompatible` ← 接口 [ServiceCfg](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/ServiceCfg.md).list
