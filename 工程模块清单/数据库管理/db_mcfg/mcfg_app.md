---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# mcfg_app

- 数据库：`db_mcfg`
- 对象类型：表

## 用途

openapi 接入方配置表：apiKey/secretKey/ivKey、IP 白名单、绑定企业、角色集合。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `mcfg_app`  (
                             `id` int(11) NOT NULL AUTO_INCREMENT,
                             `app_name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
                             `api_key` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
                             `app_config` text CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT 'app相关配置',
                             `secret_key` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
                             `iv_key` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
                             `ips` text CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                             `bind_eid` int(11) NOT NULL,
                             `roles` text CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '角色配置[1,3,4]',
                             `description` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT '',
                             `status` tinyint(4) NOT NULL DEFAULT 0,
                             `createtime` bigint(20) NOT NULL,
                             `updatetime` bigint(20) NOT NULL,
                             PRIMARY KEY (`id`) USING BTREE,
                             UNIQUE INDEX `apikey`(`api_key`) USING BTREE,
                             UNIQUE INDEX `appname`(`app_name`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `McfgAppDAOImpl`（JDBC）← `AppServiceImpl` ← 接口 [AppCfg](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/AppCfg.md)（add/delete/maintain/get/list）
