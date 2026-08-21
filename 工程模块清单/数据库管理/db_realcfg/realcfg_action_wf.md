---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_action_wf

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

action 工作流配置表：按模块配置 action 的审批/工作流。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_action_wf`  (
                                      `id` int(11) NOT NULL AUTO_INCREMENT,
                                      `module_id` int(11) NOT NULL,
                                      `api_action` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                      `api_op` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
                                      `api_keys` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '.username,.age',
                                      `min_len` int(11) NOT NULL DEFAULT 0 COMMENT '最小长度',
                                      `max_len` int(11) NOT NULL DEFAULT 0 COMMENT '最大长度',
                                      `rules` varchar(1024) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '取值规则：[]',
                                      `reversal_rules` varchar(1024) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '取值反向规则：[]',
                                      `common_rules_status` int(11) NOT NULL,
                                      `status` int(11) NOT NULL,
                                      `createtime` bigint(20) NOT NULL,
                                      `updatetime` bigint(20) NOT NULL,
                                      PRIMARY KEY (`id`) USING BTREE,
                                      INDEX `module_id`(`module_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgActionWfDAOImpl`（JDBC）← `ApiServiceImpl.listByActionWf` ← 接口 [ApiActionCfg](../../平台配置（real-cfg）/07-开放接口文档/业务规则与界面/ApiActionCfg.md)
