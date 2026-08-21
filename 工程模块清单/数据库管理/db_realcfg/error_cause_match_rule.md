---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# error_cause_match_rule

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

问题原因匹配规则表：报错文本到原因类型的匹配规则。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `error_cause_match_rule` (
    `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `error_cause_type_id` int(11) DEFAULT NULL COMMENT '问题类型的id',
    `rule_type` int(11) DEFAULT NULL COMMENT '规则类型，1为包含数据',
    `rule_msg` varchar(1024) DEFAULT NULL COMMENT '规则匹配信息',
    `create_time` datetime DEFAULT NULL COMMENT '创建时间',
    `update_time` datetime DEFAULT NULL COMMENT '更新时间',
    `status` int(11) DEFAULT NULL COMMENT '状态',
    `error_cause_log_id` int(11) DEFAULT NULL COMMENT '错误原因id',
    PRIMARY KEY (`id`),
    KEY `idx_cause_type_id_cause_log_id` (`error_cause_type_id`,`error_cause_log_id`) USING BTREE,
    KEY `idx_cause_log_id` (`error_cause_log_id`) USING BTREE
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

## 索引

- `PRIMARY KEY (`id`)`
- `KEY `idx_cause_type_id_cause_log_id` (`error_cause_type_id`,`error_cause_log_id`) USING BTREE`
- `KEY `idx_cause_log_id` (`error_cause_log_id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `ErrorCauseMatchRuleMapper`（MyBatis）：insertBatch、deleteByErrorCauseLogId、deleteByErrorCauseTypeId、selectByErrorCauseLogIds、selectAllErrorCauseMatchRule
- 经 `ErrorCauseLogService` / `ErrorCauseTypeService` 被 [ErrorCauseLogController](../../平台配置（real-cfg）/07-开放接口文档/基础设施与问题管理/ErrorCauseLogController.md)、[ErrorCauseTypeController](../../平台配置（real-cfg）/07-开放接口文档/基础设施与问题管理/ErrorCauseTypeController.md) 使用
