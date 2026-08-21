---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# sync_time_config

## 用途

同步时间配置表。SyncTimeConfigMapper/MyBatis 操作。存储时间同步相关配置。

## 所属数据库

db_realcfg

## DDL

```sql
CREATE TABLE `sync_time_config` (
                                    `id` int(11) NOT NULL AUTO_INCREMENT,
                                    `eid` int(11) DEFAULT NULL COMMENT '企业id',
                                    `ip` varchar(20) DEFAULT NULL COMMENT 'ip地址',
                                    `client_type` varchar(255) DEFAULT NULL COMMENT '客户端类型:1:服务器 2上位机',
                                    `is_server` int(1) NOT NULL DEFAULT '0' COMMENT '是否是标准时间服务器 0不是 1是',
                                    `client_os_name` int(1) DEFAULT NULL COMMENT '操作系统:1windows 2 mac 3 linux',
                                    `status` int(1) DEFAULT NULL COMMENT '是否同步成功 1失败 2成功',
                                    `last_time` bigint(30) DEFAULT NULL COMMENT '最近一次同步时间',
                                    `client_user` varchar(255) DEFAULT NULL COMMENT 'linux服务器用户',
                                    `client_password` varchar(255) DEFAULT NULL COMMENT 'linux服务器用户密码',
                                    `is_delete` int(1) NOT NULL DEFAULT '0' COMMENT '逻辑删除 0未删除 1已删除',
                                    `create_time` bigint(30) DEFAULT NULL COMMENT '创建时间',
                                    `update_time` bigint(30) DEFAULT NULL COMMENT '更新时间',
                                    PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=65 DEFAULT CHARSET=utf8mb4 COMMENT='时间同步配置表';
```

> DDL 来源：pocinit/src/mysql/db_realcfg.sql（命中）

## 设备控制中心 中的使用

同步时间配置表。SyncTimeConfigMapper/MyBatis 操作。存储时间同步相关配置。
