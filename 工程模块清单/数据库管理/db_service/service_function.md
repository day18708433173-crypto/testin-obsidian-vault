---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# service_function

- 数据库：`db_service`
- 对象类型：表

## 用途

功能菜单表：平台菜单/功能树（config_key 标识功能，parent_id 组织层级，parent_id=-1 为附属子操作）。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `service_function`  (
                                     `id` int(11) NOT NULL,
                                     `module_id` int(11) NOT NULL COMMENT '模块ID',
                                     `parent_id` int(11) NOT NULL COMMENT '父节点编号，0-表示目录根节点',
                                     `name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '菜单名称',
                                     `display_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '显示菜单名称，页面优先显示此菜单',
                                     `href` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '链接地址',
                                     `target` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '是否新窗口打开',
                                     `position` int(4) NULL DEFAULT NULL,
                                     `icon` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '菜单项对应的图标',
                                     `status` smallint(2) NULL DEFAULT NULL,
                                     `createtime` bigint(20) NULL DEFAULT NULL,
                                     `updatetime` bigint(20) NULL DEFAULT NULL,
                                     `config_key` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '配置资源key，供页面渲染识别',
                                     `parent_config_key` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '父节点的configkey',
                                     PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RoleFunctionDAOImpl`（JDBC）← `ScfgServiceImpl` ← 接口 [ServiceCfg](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/ServiceCfg.md)（list/listFunction/listFirstFunction/listRoleFunction/listRoleControl/openCloseFunction）
- 被视图 [view_role_function](view_role_function.md) 引用
