---
branch: syy.release.z7.8.1.0
module: real-scheduling
type: SQL表
database: db_task
---

# device_task_count

设备任务计数表。记录每个设备上等待执行的子子任务数量，用于任务队列监控和调度决策。

## DDL

```sql
CREATE TABLE `device_task_count` (
    `deviceid` varchar(255) NOT NULL COMMENT '设备id',
    `task_num` int(11) NULL DEFAULT 0 COMMENT '待执行子子任务数',
    `createtime` bigint(20) NULL DEFAULT NULL COMMENT '创建时间',
    `updatetime` bigint(20) NULL DEFAULT NULL COMMENT '修改时间',
    `type` varchar(255) NULL DEFAULT NULL COMMENT '任务资源端(app/web/pc)',
    PRIMARY KEY (`deviceid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 字段说明

| 字段 | 说明 |
|------|------|
| deviceid | 主键，设备 ID |
| task_num | 该设备上等待执行的子子任务数量 |
| type | 资源类型（app/web/pc） |

## 任务调度服务 中的使用

- **IDeviceTaskCountDAO**（`cn.testin.dao.interfaces.task.IDeviceTaskCountDAO`）：
  - `deviceTaskList(conditionMap)`: 按 type/deviceids 查询设备任务计数列表
- **核心流程**：
  - `Task.queueDetails` -> `IDeviceTaskCountDAO.deviceTaskList` -> 返回各设备待执行任务数
  - `DeviceTaskCountThread`（`cn.testin.schedule.task.DeviceTaskCountThread`）定时统计更新
- **POJO**：`cn.testin.pojo.task.DeviceTaskCount`
