---
branch: syy.release.z7.8.1.0
module: real-scheduling
type: SQL表
database: db_task
---

# task_sub_info

子子任务表（对应脚本）。每个子子任务代表某个设备上的某个子任务中的一个脚本执行单元。

## DDL

```sql
CREATE TABLE `task_sub_info` (
    `req_id` varchar(32) NOT NULL COMMENT '请求id，用于防止重复操作',
    `sub_subtaskid` varchar(32) NOT NULL COMMENT '子子任务（某台终端上的子任务执行的某条脚本）',
    `subtaskid` varchar(32) NOT NULL COMMENT '子任务（任务针对某台终端）',
    `script_no` int(11) NULL DEFAULT NULL COMMENT '脚本编号',
    `order_num` int(11) NOT NULL COMMENT '脚本顺序编号',
    `script_url` varchar(1024) NULL DEFAULT NULL COMMENT '脚本url',
    `script_md5` varchar(32) NULL DEFAULT NULL COMMENT '脚本md5',
    `script_type` int(11) NULL DEFAULT NULL COMMENT '脚本类型 1=iTestin 2=robotium',
    `exec_status` int(1) NOT NULL COMMENT '执行状态: 0待执行 1执行中 2预完成 3完成 4取消',
    `result_code` int(11) NULL DEFAULT NULL COMMENT '结果编码',
    `result_msg` varchar(128) NULL DEFAULT NULL COMMENT '结果信息',
    `createtime` bigint(20) NOT NULL COMMENT '创建时间',
    `precompletetime` bigint(20) NOT NULL DEFAULT 0 COMMENT '任务预完成时间',
    `updatetime` bigint(20) NOT NULL COMMENT '最后更新时间',
    `standard` varchar(254) NULL DEFAULT NULL,
    `scriptid` int(11) NULL DEFAULT NULL COMMENT '脚本id',
    `params` mediumtext NULL COMMENT '参数集',
    `param` text NULL COMMENT '参数',
    `assign_params` text NULL COMMENT '赋值全局变量情况',
    `original_ordernum` int(11) NULL DEFAULT NULL COMMENT '多端执行脚本编排顺序',
    `taskid` varchar(32) NULL DEFAULT NULL COMMENT '任务id',
    `cross_params` text NULL COMMENT '多端任务全局参数',
    `runinfo` text NULL COMMENT '多端任务运行时信息',
    `report_status` smallint(6) NULL DEFAULT NULL COMMENT '是否上报过多端 0:未 1:已上报',
    `cross_taskid` varchar(32) NULL DEFAULT NULL COMMENT '多端任务id',
    `input_params` longtext NULL COMMENT '输入变量',
    `output_params` longtext NULL COMMENT '输出变量',
    `uuid` varchar(100) NULL DEFAULT NULL COMMENT '脚本uuid',
    `row_id` int(11) DEFAULT NULL COMMENT '数据源对应的rowId',
    PRIMARY KEY (`sub_subtaskid`),
    INDEX `req_id`(`req_id`),
    INDEX `subtaskid`(`subtaskid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='子子任务表，对应脚本';
```

## 字段说明

| 字段 | 说明 |
|------|------|
| sub_subtaskid | 主键，全网唯一子子任务 ID |
| subtaskid | 关联的子任务 ID（task_info） |
| order_num | 脚本执行排序，数值从小到大依次执行 |
| original_ordernum | 跨端任务的原始编排顺序 |
| script_no | 脚本编号 |
| script_url/script_md5/script_type | 脚本下载地址/MD5/类型 |
| exec_status | 0待执行/1执行中/2预完成/3完成/4取消 |
| input_params/output_params | 脚本执行前后的全局/局部变量 |
| cross_params | 跨端任务从前一终端传递的全局参数 |
| report_status | 跨端任务执行后上报状态 |

## 任务调度服务 中的使用

- **ITaskSubInfoDAO**（`cn.testin.dao.interfaces.task.ITaskSubInfoDAO`）：
  - `batchInsert`: 初始化时批量写入脚本列表（每批最多20条）
  - `list(conditionMap, sortFields)`: 按 subtaskid 查询子子任务列表
  - `deleteByTaskId(taskid)`: 按任务 ID 删除
  - `deleteByReqId(reqId)`: 按请求 ID 删除
  - `update(subInfo)`: 更新执行状态/结果/变量
  - `sortScriptsByOrderNum(subtaskid, deviceid)`: 按 orderNum 排序获取脚本列表
- **核心流程**：
  - `Task.init` -> batchInsert（初始化）
  - `Task.precomplete` -> UPDATE exec_status=2（预完成）+ inputParams/outputParams
  - `Task.reportResult` -> UPDATE exec_status=3（完成）
  - `cross.Task.execute` -> 按 originalOrderNum 查询下一组子子任务，设置 crossParams
  - `ucomInterruptTask` -> 查询 subtaskid 下全部子子任务，逐个上报异常结果
- **POJO**：`cn.testin.pojo.task.DbTaskSubInfo`
- **视图**：`view_sub_subtask_info`（task_sub_info JOIN task_info）
