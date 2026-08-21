---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# error_cause_operate_log

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

问题原因操作日志表：原因配置的新增/修改/删除操作流水。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `error_cause_operate_log` (
                                           `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
                                           `task_id` varchar(100) DEFAULT NULL COMMENT '任务id',
                                           `sub_task_id` varchar(100) DEFAULT NULL COMMENT '子任务id',
                                           `sub_sub_task_id` varchar(100) DEFAULT NULL COMMENT '子子任务id',
                                           `script_no` bigint(20) DEFAULT NULL COMMENT '脚本编号',
                                           `error_cause_type_id` bigint(20) DEFAULT NULL COMMENT '错误类型id',
                                           `category` int(3) DEFAULT NULL COMMENT '系统自定义类型',
                                           `error_cause` varchar(255) DEFAULT NULL COMMENT '错误原因',
                                           `error_report` tinyint(1) DEFAULT NULL COMMENT '是否误报 0-否 1-是',
                                           `create_user_id` bigint(20) DEFAULT NULL COMMENT '创建人',
                                           `create_time` datetime DEFAULT NULL COMMENT '创建时间',
                                           `update_time` datetime DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
                                           `status` tinyint(4) DEFAULT NULL COMMENT '状态  1-有效   0-无效',
                                           PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;
```

## 索引

- `PRIMARY KEY (`id`)`

## 被哪些接口/mapper 方法使用

- `ErrorCauseOperateLogMapper`（MyBatis）：selectByCondition、insertOperateLog、updateById
- 经 `ErrorCauseOperateLogService` 被接口 [ErrorCauseOperateLogController](../../平台配置（real-cfg）/07-开放接口文档/基础设施与问题管理/ErrorCauseOperateLogController.md) 使用
- `OperateLogStrategy`（business.strategy.log）：操作日志先入 Redis 队列 `queue:operate_log` 后异步落库
