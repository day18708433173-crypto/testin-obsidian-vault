---
branch: syy.release.z7.8.1.0
module: real-scheduling
type: SQL表
database: db_task
---

# task_script_device_focus

任务脚本设备聚焦表。用于将某个任务中的某条脚本聚焦到指定设备上执行，支持匹配次数控制，防止脚本被重复分配到同一设备。

## DDL

```sql
CREATE TABLE `task_script_device_focus` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `task_id` varchar(100) DEFAULT NULL COMMENT '任务id',
    `script_no` int(11) DEFAULT NULL COMMENT '脚本编号',
    `device_id` varchar(100) DEFAULT NULL COMMENT '聚焦到的设备id',
    `create_time` datetime DEFAULT NULL COMMENT '创建时间',
    `task_type` int(11) DEFAULT NULL COMMENT '任务类型',
    `match_count` int(11) DEFAULT NULL COMMENT '匹配次数',
    PRIMARY KEY (`id`),
    KEY `idx_task_id_script_no` (`task_id`, `script_no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

## MyBatis Mapper

```xml
<!-- TaskScriptDeviceFocusMapper.xml, namespace: cn.testin.dao.impl.task.TaskScriptDeviceFocusDAO -->
<update id="updateMatchCount"> UPDATE ... SET match_count = match_count + (#{matchCount}) WHERE id = #{id}</update>
<delete id="deleteByTaskId"> DELETE WHERE task_id = #{taskId}</delete>
<delete id="deleteByIdAndMathCount"> DELETE WHERE id = #{id} AND match_count = #{matchCount}</delete>
```

## 任务调度服务 中的使用

- **TaskScriptDeviceFocusDAO**（`cn.testin.dao.impl.task.TaskScriptDeviceFocusDAO`，使用 MyBatis Mapper）：
  - `updateMatchCount`: 增加/减少匹配计数（+1 或 -1）
  - `deleteByTaskId(taskId)`: 按任务 ID 清除
  - `deleteByIdAndMathCount(id, matchCount)`: CAS 方式按版本号删除
- **业务层**：`TaskScriptDeviceFocusService`（`cn.testin.mvc.service.TaskScriptDeviceFocusService`）
  - `selectWithoutDeviceId(deviceId, limit)`: 查询当前设备不在聚焦列表中的任务
  - `getDbTaskScriptDeviceFocusByTaskIdAndSubTaskId(taskId, subTaskId)`: 查询指定任务子任务的聚焦配置
  - `deleteByIdAndMathCount(id, matchCount)`: 删除指定匹配次数的记录（乐观锁）
  - `updateMatchCount(record)`: 更新匹配计数
- **核心流程**：
  - `Task.prematch` -> 查询 selectWithoutDeviceId，优先匹配未聚焦到当前设备的脚本
  - 预匹配失败时（超过2分钟） -> match_count -= 1 或 deleteByIdAndMathCount（归还聚焦名额）
  - 聚焦机制确保同一条脚本不会在同一个设备上重复执行
- **POJO**：`cn.testin.pojo.task.DbTaskScriptDeviceFocus`
