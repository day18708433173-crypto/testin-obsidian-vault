---
branch: syy.release.z7.8.1.0
module: real-test
type: SQL
database: MySQL (db_real)
table: quartz_job_statement
---

# quartz_job_statement

定时任务执行流水表，记录每次定时任务调度产生的 taskId，按 taskId 可删除关联记录。

## DDL

```sql
CREATE TABLE `quartz_job_statement` (
  `statement_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '流水单ID',
  `job_id` int(11) NOT NULL COMMENT '定时任务ID',
  `task_id` varchar(45) DEFAULT NULL COMMENT '提测任务ID',
  `create_time` bigint(20) DEFAULT NULL COMMENT '流水创建时间',
  `update_time` bigint(20) DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`statement_id`),
  UNIQUE KEY `statement_id_UNIQUE`(`statement_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=Compact;
```

## app处理服务 中的使用

### DAO/Service
- DAO: `QuartzJobStatementDAOImpl` -> `IQuartzJobStatementDAO`
- Mapper: `QuartzJobStatementMapper.xml`
- Service: `QuartzJobStatementService.removeByTaskId`, `ScheduledJob`

### 关键SQL

| 操作 | SQL | 说明 |
| --- | --- | --- |
| 查询 | SELECT * FROM quartz_job_statement WHERE task_id = #{taskId} | 根据 taskId 查记录 |
| 删除 | DELETE FROM quartz_job_statement WHERE task_id = #{taskId} | 根据 taskId 删除(取消任务时调用) |
