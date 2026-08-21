---
branch: syy.release.z7.8.1.0
module: real-scheduling
type: SQL表
database: db_task
---

# task_abnormal_device

异常设备记录表。记录需要延迟撤销/处理的异常设备，由 AbnormalDeviceHandlerThread 定时扫描处理。

## DDL

```sql
CREATE TABLE `task_abnormal_device` (
    `deviceid` varchar(128) NOT NULL,
    `ucomid` varchar(128) NOT NULL,
    `vhost` int(11) NOT NULL,
    `content` text NULL,
    `status` int(11) NOT NULL,
    `createtime` bigint(20) NOT NULL,
    `updatetime` bigint(20) NOT NULL,
    `publishtime` bigint(20) NOT NULL,
    PRIMARY KEY (`deviceid`),
    INDEX `pending`(`vhost`, `publishtime`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 字段说明

| 字段 | 说明 |
|------|------|
| deviceid | 主键，异常设备 ID（WEB端即为ucomid） |
| ucomid | 上位机 ID |
| vhost | 模块节点 ID |
| publishtime | 待处理时间，超过此时间后由定时线程处理 |
| status | 处理状态 |
| content | 异常内容/上下文信息 |

## 任务调度服务 中的使用

- **ITaskAbnormalDeviceDAO**（`cn.testin.dao.impl.task.TaskAbnormalDeviceDAOImpl`）：
  - `add(abnormalDevice)`: 新增异常设备记录（revoke 带延迟时写入）
  - `remove(deviceid)`: 移除异常设备（通过 AbnormalDevice.remove 接口）
  - `list(vhost, max)`: 查询待处理的异常设备列表
- **核心流程**：
  - `Task.revoke`（带 expirePeriod） -> IAbnormalDeviceService.add() -> INSERT
  - `AbnormalDeviceLoadThread` -> 定时查询 publishtime <= now 的记录
  - `AbnormalDeviceHandlerThread` -> 执行撤销回收逻辑，完成后 DELETE
  - `AbnormalDevice.remove` -> 手动移除异常设备（DELETE）
- **定时线程**：
  - `AbnormalDeviceLoadThread`：加载待处理异常设备到队列
  - `AbnormalDeviceDispatchThread`：分发到工作线程
  - `AbnormalDeviceHandlerThread`：执行回收处理
- **POJO**：`cn.testin.pojo.task.DbTaskAbnormalDevice`
