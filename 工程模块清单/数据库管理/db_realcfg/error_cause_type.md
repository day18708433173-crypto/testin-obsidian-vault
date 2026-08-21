---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# error_cause_type

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

问题原因类型表：报错原因的分类定义。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `error_cause_type` (
                                    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
                                    `eid` int(11) DEFAULT '1' COMMENT '企业id',
                                    `project_id` int(11) DEFAULT NULL COMMENT '项目id',
                                    `name` varchar(255) DEFAULT NULL COMMENT '类型名称',
                                    `desc` varchar(255) DEFAULT NULL COMMENT '备注',
                                    `error_report` tinyint(1) DEFAULT '0' COMMENT '误报 0-关闭 1-开启',
                                    `create_user_id` int(11) DEFAULT NULL COMMENT '创建人',
                                    `create_user_name` varchar(255) DEFAULT NULL COMMENT '创建人名称',
                                    `update_user_id` int(11) DEFAULT NULL COMMENT '更新人',
                                    `update_user_name` varchar(255) DEFAULT NULL COMMENT '跟新人名称',
                                    `create_time` datetime DEFAULT NULL COMMENT '创建时间',
                                    `update_time` datetime DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
                                    `status` tinyint(1) DEFAULT '1' COMMENT '状态  1-正常  0-删除',
                                    PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=101 DEFAULT CHARSET=utf8mb4;
```

## 索引

- `PRIMARY KEY (`id`)`

## 被哪些接口/mapper 方法使用

- `ErrorCauseTypeMapper`（MyBatis）：selectCount、insertBatchErrorCause、selectErrorCauseTypeByCondition、updateErrorCauseTypeById、countByCondition、selectErrorCauseTypeById、selectMaxId、saveOne、selectErrorCauseTypeList
- 经 `ErrorCauseTypeService` 被接口 [ErrorCauseTypeController](../../平台配置（real-cfg）/07-开放接口文档/基础设施与问题管理/ErrorCauseTypeController.md) 使用
