---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# view_env_project

- 数据库：`db_realcfg`
- 对象类型：视图

## 用途

环境-项目关联视图：realcfg_env_config JOIN realcfg_env_enterprise。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE VIEW `view_env_project` AS select `envConfig`.`id` AS `env_id`,`envConfig`.`eid` AS `env_eid`,`envConfig`.`name` AS `env_name`,`envConfig`.`hosts` AS `env_hosts`,`envConfig`.`db_config` AS `env_dbconfig`,`envConfig`.`descr` AS `env_descr`,`envConfig`.`status` AS `env_status`,`envProject`.`project_id` AS `env_projectid`,`envProject`.`project_name` AS `env_projectname` from (`realcfg_env_config` `envConfig` join `realcfg_env_enterprise` `envProject` on((`envConfig`.`id` = `envProject`.`env_id`)));
```

## 索引

- 视图，无索引

## 被哪些接口/mapper 方法使用

- `EnvMapper`（MyBatis）：getEnvListByCondition ← `EnvService`
- pojo `RealCfgEnvConfigView` 映射该视图
