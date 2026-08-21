---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_db_config

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

数据源配置表：登记 MySQL/Redis/MongoDB 等数据源连接信息（环境配置引用）。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `realcfg_db_config` (
                                     `id` int(11) NOT NULL AUTO_INCREMENT,
                                     `eid` int(11) NOT NULL,
                                     `type_id` int(11) NOT NULL,
                                     `db_address` varchar(256) NOT NULL,
                                     `db_port` varchar(6) NOT NULL,
                                     `db_name` varchar(64) DEFAULT NULL,
                                     `db_user` varchar(64) NOT NULL,
                                     `db_password` varchar(64) NOT NULL,
                                     `db_secret_pwd` varchar(500) DEFAULT NULL COMMENT '密码加密后的密文',
                                     `db_descr` varchar(256) DEFAULT NULL,
                                     `timeout` int(4) unsigned NOT NULL DEFAULT '10' COMMENT '超时时间',
                                     `status` tinyint(4) NOT NULL DEFAULT '1',
                                     `createtime` bigint(20) NOT NULL,
                                     `updatetime` bigint(20) DEFAULT NULL,
                                     `project_id` int(11) DEFAULT '0',
                                     `project_name` varchar(255) DEFAULT NULL,
                                     `env_id` int(11) DEFAULT NULL COMMENT '所属环境id',
                                     `channel` int(11) DEFAULT NULL COMMENT '1：环境管理创建  0：数据库管理创建',
                                     `db_alias` varchar(255) DEFAULT NULL COMMENT '唯一标识',
                                     PRIMARY KEY (`id`) USING BTREE,
                                     KEY `eid_type_id` (`eid`,`type_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=15 DEFAULT CHARSET=utf8 ROW_FORMAT=COMPACT COMMENT='数据库配置表';
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`
- `KEY `eid_type_id` (`eid`,`type_id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `RealcfgDbConfigDAOImpl`（JDBC）← `RealCfgDbConfigServiceImpl` ← 接口 [DbCfg](../../平台配置（real-cfg）/07-开放接口文档/数据源与代码配置/DbCfg.md)
