# QuartzLogController — 定时任务日志删除

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/mvc/controller/QuartzLogController.java`
> 类级路由：`/schedule_log`

## 接口列表

| # | 方法 | 路径 | 方法名 | 说明 |
|---|---|---|---|---|
| 1 | DELETE | `/v3/schedule_log/schedule_log/remove_by_task_id` | removeByTaskId | 按任务ID删除定时日志 |

统一响应包装：`ResponseResult<Integer> { int code; String msg; Integer data }`。

---

## 1. DELETE /v3/schedule_log/schedule_log/remove_by_task_id — 按任务ID删除定时日志

### 入口

`QuartzLogController.removeByTaskId(@RequestParam("task_id") String taskId)`

### 请求参数

| 字段 | 来源 | 必填 | 说明 |
|------|------|------|------|
| task_id | Query | 是 | 任务ID |

### 响应结构

`ResponseResult<Integer>`，data = 删除行数（0 表示无记录）。

### 实现意图

查询 `quartz_job_log` 表中 `task_id` 匹配的记录，若存在则删除。返回删除行数。若不存在记录直接返回 0。

### 调用链

```
QuartzLogController.removeByTaskId
└─ QuartzLogService.removeByTaskId(taskId)
   ├─ QuartzJobLogMapper.selectList(where task_id=?) → MySQL quartz_job_log
   └─ QuartzJobLogMapper.delete(where task_id=?) → MySQL quartz_job_log
```

### 涉及表

| 存储 | 表 | 操作 |
|------|-----|------|
| MySQL db_common | quartz_job_log | 读 + 删 |

### 注意

路径出现双重 `/schedule_log/schedule_log/`：类级 `@RequestMapping("schedule_log")` + 方法级 `@DeleteMapping("/schedule_log/remove_by_task_id")`，最终为 `/v3/schedule_log/schedule_log/remove_by_task_id`。

---

## 备注

- 仅一个端点，极简 Controller。
- 无 `@Valid`、`@OperateLog`、`@Transactional`。
- 多次调用幂等（已删除则返回 0）。

相关文档：[00-分支索引](00-分支索引.md) · [HeartBeatController](HeartBeatController.md)
