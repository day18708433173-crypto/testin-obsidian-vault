---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# mcfg_api

- 数据库：`db_mcfg`
- 对象类型：表

## 用途

openapi 接口元数据表：每个 action/op 一条记录，归属模块。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `mcfg_api`  (
                             `id` int(11) NOT NULL,
                             `module_id` int(11) NOT NULL,
                             `api_action` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                             `api_op` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                             `special_api_action` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                             `special_api_op` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
                             `descr` varchar(254) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
                             `need_login` tinyint(4) NOT NULL DEFAULT 1 COMMENT '是否需要登录使用',
                             `need_activation` tinyint(4) NOT NULL DEFAULT 1 COMMENT '否是激活才能使用',
                             `need_write` tinyint(4) NOT NULL COMMENT '是否有写库的操作',
                             `protocol_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '赋值配置：\"additionalKeys\":{\"name\":[\"key1\",\"key2\",\"key3\",\"key4\"],\"objInfo\":{\"name\":\"qianggao\"}},\"ignoreKeys|fieldKeys|checkKeys\":{\"info\":{\"name\",\"obj\":{\"k1\",\"k2\"}}}',
                             `rule_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '暂不启用',
                             `access_frequency` int(11) NOT NULL DEFAULT 0 COMMENT '0 默认不限;
```

## 索引

- DDL 中未见显式索引

## 被哪些接口/mapper 方法使用

- `McfgApiDAOImpl`（JDBC）← `ApiServiceImpl` ← 接口 [ApiCfg](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/ApiCfg.md).list
- `RoleServiceImpl.addApiList` 中做 apiId 存在性校验（接口 [RoleCfg](../../平台配置（real-cfg）/07-开放接口文档/用户与权限/RoleCfg.md).addApi）
