---
branch: syy.release.z7.8.1.0
module: real-scheduling
type: SQL表
database: db_task
---

# task_release_time_periods

任务下发时间段控制表。支持三种类型：1=永久暂停、2=时间段内恢复下发、3=时间段内暂停下发。用于灵活控制任务的调度窗口。

## DDL

```sql
CREATE TABLE `task_release_time_periods` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '自增id',
    `taskId` varchar(32) NOT NULL COMMENT '关联的任务id',
    `start_time` bigint(13) DEFAULT NULL COMMENT '开始时间',
    `end_time` bigint(13) DEFAULT NULL COMMENT '结束时间',
    `type` smallint(2) NOT NULL COMMENT '类型：1暂停 2时间段内下发 3时间段内停止',
    `create_time` bigint(13) NOT NULL COMMENT '创建时间',
    PRIMARY KEY (`id`),
    KEY `taskId_index` (`taskId`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='存放任务下发时间段的表';
```

## 字段说明

| 字段 | 说明 |
|------|------|
| taskId | 关联的任务 ID |
| type | 1=STOP（永久暂停）、2=TIMEPERIODS（时间段内下发）、3=TIMEPERIODSSTOP（时间段内暂停） |
| start_time / end_time | 时间段范围（ms），type=1时可为null |

## MyBatis Mapper

```xml
<!-- TaskMapper.xml, namespace: cn.testin.dao.impl.task.TaskReleaseTimePeriodsInfoDAO -->
<insert id="addBatch"> INSERT INTO task_release_time_periods (...) VALUES ...</insert>
<select id="getPauseTaskIds">
    查询 type=1 或 (type=3 且在时间段内) 或 (不在 type=2 恢复时间段内) 的任务
</select>
<select id="getResumeTaskIds">
    查询 type=2 且当前在时间段内的任务
</select>
```

## 任务调度服务 中的使用

- **TaskReleaseTimePeriodsInfoDAO**（`cn.testin.dao.impl.task.TaskReleaseTimePeriodsInfoDAO`，使用 MyBatis Mapper）：
  - `insert(period)`: 新增单条记录
  - `addBatch(list)`: 批量新增（任务初始化时）
  - `delete(period)`: 按 taskId 删除所有记录
  - `getCount(taskId)`: 统计 taskId 下的记录数
  - `getPauseTaskIds(time, taskType)`: 查询当前应暂停的任务 ID 列表
  - `getResumeTaskIds(time, taskType)`: 查询当前应恢复的任务 ID 列表
- **MVC 层**：`TaskReleaseTimePeriodsServiceImpl`（`cn.testin.mvc.service.TaskReleaseTimePeriodsServiceImpl`）
  - `pauseTask`: INSERT type=STOP + 调用 TestApi 暂停 + 写 operate_log
  - `resumeTask`: DELETE 所有记录 + 调用 TestApi 恢复 + 写 operate_log
  - `getPauseTaskIds/getResumeTaskIds`: 按当前时间 HHmmss 查询应控制的任务
- **调度线程**：`TaskCancelCheckThread` 定时调用 getPauseTaskIds/getResumeTaskIds 控制任务下发
- **POJO**：`cn.testin.pojo.task.TaskReleaseTimePeriods`
