---
branch: syy.release.z7.8.1.0
module: real-scheduling
type: SQL表
database: db_task
---

# task_exec_relation

设备与任务的执行关系表。记录设备当前正在执行（或预匹配占用）的任务信息，是任务匹配防重和回收的关键表。

## DDL

```sql
CREATE TABLE `task_exec_relation` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT,
    `deviceid` varchar(128) NOT NULL,
    `vhost` int(11) NOT NULL,
    `ucomid` varchar(128) NOT NULL,
    `taskid` varchar(32) NOT NULL,
    `subtaskid` varchar(32) NOT NULL,
    `exec_status` int(11) NOT NULL,
    `network_simulation` int(11) NOT NULL COMMENT '模拟网络',
    `version` bigint(20) NOT NULL,
    `status` int(11) NOT NULL,
    `createtime` bigint(20) NOT NULL,
    `updatetime` bigint(20) NOT NULL,
    `expiretime` bigint(20) NOT NULL,
    PRIMARY KEY (`id`),
    UNIQUE INDEX `subtaskid`(`subtaskid`),
    UNIQUE INDEX `deviceid_execstatus`(`deviceid`, `exec_status`),
    INDEX `simulation`(`network_simulation`, `ucomid`),
    INDEX `expire_vhost`(`expiretime`, `vhost`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 字段说明

| 字段 | 说明 |
|------|------|
| subtaskid | 唯一索引，一个子任务在关系表中只能有一条记录 |
| deviceid_execstatus | 唯一索引，一个设备同一执行状态只能有一条记录 |
| exec_status | IDLE(空闲/预匹配) / ING(执行中) |
| network_simulation | 是否为模拟网络任务 |
| expiretime | 预匹配关系过期时间（默认2分钟），过期后关系作废可被重新匹配 |

## 任务调度服务 中的使用

- **ITaskExecRelationDAO**（`cn.testin.dao.interfaces.task.ITaskExecRelationDAO`）：
  - `list(deviceid, execStatuses)`: 查询设备当前的执行关系
  - `insert(relation)`: 预匹配时创建占用关系
  - `delete(subtaskid)`: 任务回收/完成时删除关系
  - `update(relation)`: 更新执行状态
- **核心流程**：
  - `Task.prematch` -> INSERT exec_status=IDLE（预匹配占用，120秒过期）
  - `Task.match` (确认) -> UPDATE exec_status=ING（执行中）
  - `Task.recover` -> 检查过期关系，回收执行中任务
  - 设备 deviceid_execstatus 唯一约束保证同一设备只能执行一个任务
  - subtaskid 唯一约束保证同一子任务不能被多个设备同时领取
- **Redis 缓存**：`DeviceTaskRelationDAOImpl`（`cn.testin.dao.impl.redis.DeviceTaskRelationDAOImpl`）在 Redis 中以 Hash 结构同步存储设备-任务关系
- **POJO**：`cn.testin.pojo.task.DbTaskExecRelation`
