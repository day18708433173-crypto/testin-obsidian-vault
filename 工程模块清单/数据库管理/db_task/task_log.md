---
branch: syy.release.z7.8.1.0
module: real-scheduling
type: SQL表
database: db_task
---

# task_log

任务流转日志表。记录任务生命周期中的每一次状态变更（匹配、预完成、结果上报、回收、取消、过期等）。

## DDL

```sql
CREATE TABLE `task_log` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '自增id',
    `taskid` varchar(32) NOT NULL COMMENT '关联的任务id，对应APP',
    `subtaskid` varchar(32) NOT NULL COMMENT '子任务id，对应设备',
    `sub_subtaskid` varchar(32) NULL DEFAULT NULL COMMENT '子子任务id，对应脚本',
    `action` varchar(32) NOT NULL COMMENT '流转动作: match/preComplete/report/recover/cancel/expire',
    `deviceid` varchar(128) NULL DEFAULT NULL COMMENT '设备id',
    `detail` text NULL COMMENT '任务详细信息',
    `createtime` bigint(20) NOT NULL COMMENT '创建时间',
    PRIMARY KEY (`id`),
    INDEX `createtime`(`createtime`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='任务流转日志表，对应脚本';
```

## 字段说明

| 字段 | 说明 |
|------|------|
| action | 流转动作：match(匹配)、preComplete(预完成)、report(结果上报)、recover(回收)、cancel(取消)、expire(过期) |
| detail | 任务详细信息 JSON（状态变更前后的数据） |
| sub_subtaskid | 可为 null，任务级别日志时不传 |

## 任务调度服务 中的使用

- **DbTaskLog / DbTaskLogRowMapper**（`cn.testin.pojo.task.DbTaskLog`）：POJO 和 RowMapper
- **核心流程**：
  - `ITaskService` 的实现（TaskServiceImpl）在每个状态变更节点调用 `this.itasklogdao.insert(log)`
  - `match`: 任务匹配到设备时写入
  - `preComplete`: 设备预完成时写入
  - `report`: 结果上报时写入
  - `recover`: 任务回收时写入
  - `cancel`: 任务取消时写入
  - `expire`: 任务过期时写入
- **DAO 接口**：`ITaskLogDAO`（推断，基于 POJO 注释）
- **POJO**：`cn.testin.pojo.task.DbTaskLog`
