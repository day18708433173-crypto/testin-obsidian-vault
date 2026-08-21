---
branch: syy.release.z7.8.1.0
module: real-scheduling
type: SQL表
database: db_task
---

# task_result

子子任务结果表（对应脚本执行结果）。每条记录记录一个脚本执行的最终结果（原始文件 URL、结果编码、变量信息等），是 real-analysis 模块解析的输入。

## DDL

```sql
CREATE TABLE `task_result` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '自增主键',
    `vhost` int(11) NOT NULL DEFAULT 1040001,
    `subtaskid` varchar(32) NOT NULL COMMENT '子任务id',
    `sub_subtaskid` varchar(32) NOT NULL COMMENT '子子任务id',
    `user_id` int(11) NOT NULL COMMENT '用户id',
    `taskid` varchar(32) NOT NULL COMMENT '任务id',
    `deviceid` varchar(128) NULL DEFAULT NULL COMMENT '设备id',
    `result_url` varchar(300) NULL DEFAULT NULL COMMENT '测试结果原始未解析地址',
    `summary_url` varchar(300) NULL DEFAULT NULL COMMENT '结果文件解析后汇总地址',
    `result_code` int(11) NULL DEFAULT NULL COMMENT '结果明细编码',
    `result_msg` varchar(254) NULL DEFAULT NULL COMMENT '测试结果说明',
    `exec_num` int(4) NOT NULL DEFAULT 0 COMMENT '任务执行次数',
    `parse_num` int(4) NOT NULL DEFAULT 0 COMMENT '结果解析次数',
    `parse_time` bigint(20) NOT NULL DEFAULT 0 COMMENT '结果解析时间',
    `status` int(4) NOT NULL DEFAULT 0 COMMENT '0未解析 1已解析 2解析失败',
    `init_tasktime` bigint(20) NULL DEFAULT NULL COMMENT '初始化任务时间',
    `match_tasktime` bigint(20) NULL DEFAULT NULL COMMENT '匹配任务时间',
    `precompletetime` bigint(20) NULL DEFAULT NULL COMMENT '预完成时间',
    `createtime` bigint(20) NOT NULL COMMENT '创建时间',
    `updatetime` bigint(20) NOT NULL COMMENT '最后更新时间',
    `exec_standard` varchar(32) NULL DEFAULT NULL COMMENT '执行策略',
    `unique_key` varchar(512) NULL DEFAULT NULL COMMENT '任务全局唯一标识',
    `order_num` int(11) NULL DEFAULT NULL COMMENT '脚本顺序号',
    `round` smallint(6) NULL DEFAULT NULL COMMENT '执行轮数',
    `retry_num` smallint(6) NULL DEFAULT 0 COMMENT '重试顺序',
    `os_name` varchar(128) NULL DEFAULT NULL COMMENT '系统类型',
    `browser_type` varchar(128) NULL DEFAULT NULL COMMENT '浏览器类型',
    `browser_version` varchar(128) NULL DEFAULT NULL COMMENT '浏览器版本',
    `ucomid` varchar(128) NULL DEFAULT NULL COMMENT '上位机id',
    `biz_code` int(11) NULL DEFAULT NULL COMMENT '业务编码',
    `standard` varchar(255) NULL DEFAULT NULL COMMENT 'sid',
    `sid` varchar(255) NULL DEFAULT NULL COMMENT '浏览器session id',
    `cross_taskid` varchar(32) NULL DEFAULT NULL COMMENT '多端任务id',
    `resource_type` varchar(32) NULL DEFAULT NULL COMMENT 'app/web/pc',
    `input_params` longtext NULL COMMENT '输入变量',
    `output_params` longtext NULL COMMENT '输出变量',
    `row_id` int(11) DEFAULT NULL COMMENT '数据源行id',
    `app_md5` varchar(64) NULL COMMENT '应用包MD5',
    PRIMARY KEY (`id`),
    UNIQUE INDEX `sub_subtaskid_UNIQUE`(`sub_subtaskid`, `match_tasktime`, `retry_num`),
    INDEX `task_id`(`subtaskid`),
    INDEX `vhost_status`(`status`),
    INDEX `adapt_id`(`taskid`),
    INDEX `createtime`(`createtime`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='子子任务结果表，对应脚本';
```

## 字段说明

| 字段 | 说明 |
|------|------|
| id | 自增主键 |
| sub_subtaskid + match_tasktime + retry_num | 唯一约束，同一脚本同一次匹配执行（含重试）只有一条结果 |
| result_url | 设备上传的原始结果文件 URL（real-logfile 存储） |
| summary_url | real-analysis 解析完成后回填的汇总文件 URL |
| result_code / result_msg | 结果编码与描述 |
| status | 0未解析 / 1已解析 / 2解析失败（由 real-analysis 更新） |
| input_params / output_params | 脚本执行前后的变量（JSON） |

## 任务调度服务 中的使用

- **ITaskResultDAO**（`cn.testin.dao.interfaces.task.ITaskResultDAO`）：
  - `insert(result)`: 设备结果上报时插入
  - `update(result)`: 结果解析后更新 summary_url 等
  - `list(conditionMap, offset, limit)`: 查询结果列表
- **核心流程**：
  - `Task.reportResult` -> IResultService.report() -> INSERT task_result(status=0)
  - `ResultDispatchThread/ResultHandlerThread` -> 轮询 status=0 的结果，调用 real-analysis 解析
  - real-analysis 回调 -> UPDATE status=1/2, summary_url
  - `ucomInterruptTask` -> 直接 INSERT task_result(result_code=DEVICEEXCEPTION)
- **POJO**：`cn.testin.pojo.task.DbTaskResult`
