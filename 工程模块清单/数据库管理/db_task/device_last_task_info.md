---
branch: syy.release.z7.8.1.0
module: real-scheduling
type: SQL表
database: db_task
---

# device_last_task_info

设备最近执行的任务信息表。记录每个设备最近执行过的任务和脚本编号，用于数据驱动执行策略中判断设备最后执行的脚本位置，避免重复执行。

## DDL

```sql
CREATE TABLE `device_last_task_info` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `device_id` varchar(255) DEFAULT NULL COMMENT '设备id',
    `device_type` int(11) DEFAULT NULL COMMENT '设备类型',
    `task_id` varchar(100) DEFAULT NULL COMMENT '任务id',
    `sub_task_id` varchar(100) DEFAULT NULL COMMENT '子任务id',
    `sub_sub_task_id` varchar(100) DEFAULT NULL COMMENT '子子任务id',
    `script_no` int(11) DEFAULT NULL COMMENT '执行的脚本编号',
    `create_time` datetime DEFAULT NULL COMMENT '创建时间',
    PRIMARY KEY (`id`),
    KEY `idx_device_id_script_no` (`device_id`, `script_no`),
    KEY `idx_task_id_script_no` (`task_id`, `script_no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='设备最后执行的任务信息';
```

## MyBatis Mapper

```xml
<!-- DeviceLastTaskInfoMapper.xml, namespace: cn.testin.dao.impl.task.DeviceLastTaskInfoDAO -->
<insert id="insertBatch"> INSERT INTO device_last_task_info (...) VALUES ...</insert>
<delete id="deleteByDeviceId"> DELETE FROM device_last_task_info WHERE device_id = #{deviceId}</delete>
<select id="getLastTaskInfoByDeviceId"> SELECT * WHERE device_id = #{deviceId} AND device_type = #{deviceType} LIMIT 1</select>
<select id="getScriptNosByTaskId"> SELECT DISTINCT script_no WHERE task_id = #{taskId}</select>
```

## 任务调度服务 中的使用

- **DeviceLastTaskInfoDAO**（`cn.testin.dao.impl.task.DeviceLastTaskInfoDAO`，使用 MyBatis Mapper）：
  - `insertBatch(list)`: 批量写入设备最近任务信息
  - `deleteByDeviceId(deviceId)`: 清除设备的最近任务记录
  - `getLastTaskInfoByDeviceId(deviceId, deviceType)`: 查询设备最近执行的任务
  - `getScriptNosByTaskId(taskId)`: 查询某任务下所有设备已执行的脚本编号集合
- **业务层**：`DeviceLastTaskInfoServiceImpl`（`cn.testin.business.impl.DeviceLastTaskInfoServiceImpl`）
  - `updateDeviceLastTaskInfo(taskJson)`: 匹配任务后，当 execStandard=data 时更新设备的最后任务信息
- **核心流程**：
  - 仅当 `execStandard = "data"`（数据驱动执行策略）时使用
  - 匹配任务后记录设备的 taskId/subTaskId/subSubTaskId/scriptNo
  - 后续匹配时通过 scriptNo 判断哪些脚本已执行过，跳过已执行的脚本
- **POJO**：`cn.testin.pojo.task.DbDeviceLastTaskInfo`
