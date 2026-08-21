---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_project_advanced_config

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

项目高级配置表：按 project_id + config_type（TIMEOUT/VARRULES）保存 JSON 配置内容。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `db_realcfg`.`realcfg_project_advanced_config`( `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键', `project_id` int(11) NOT NULL COMMENT '项目id', `config_type` varchar(225) NOT NULL COMMENT '配置类型（TIMEOUT/VARRULES）', `content` longtext COMMENT '数据', `create_user_id` int(11) DEFAULT NULL COMMENT '创建人id', `update_user_id` int(11) DEFAULT NULL COMMENT '更新人id', `create_time` bigint(20) DEFAULT NULL COMMENT '创建时间', `update_time` bigint(20) DEFAULT NULL COMMENT '更新时间', PRIMARY KEY (`id`), KEY `idx_project_id` (`project_id`)) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='项目高级配置表';
```

## 索引

- DDL 中未见显式索引

## 被哪些接口/mapper 方法使用

- `ProjectAdvancedConfigDoMapper`（MyBatis，自动生成 CRUD + selectAllWithCondition）
- `ExtProjectAdvancedConfigDoMapper`（ext）：selectByProjectIdAndType
- 经 `ProjectAdvancedConfigService` 被接口 [ProjectAdvancedConfigController](../../平台配置（real-cfg）/07-开放接口文档/项目与平台配置/ProjectAdvancedConfigController.md) 使用
