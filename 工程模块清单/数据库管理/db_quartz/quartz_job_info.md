---
branch: syy.release.z7.8.1.0
module: real-test
type: SQL
database: MySQL (db_real)
table: quartz_job_info
---

# quartz_job_info

定时任务信息表，存储所有定时任务/模板的配置（cron 表达式、设备、脚本、通知等），是 app处理服务 模块定时任务管理的核心表。

## DDL

```sql
CREATE TABLE `quartz_job_info` (
  `job_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '定时任务ID',
  `job_name` varchar(45) DEFAULT NULL COMMENT '定时任务名称',
  `job_rule` varchar(300) DEFAULT '' COMMENT '任务执行规则：JSON字符串',
  `job_status` char(6) DEFAULT NULL COMMENT '状态：0未执行；1暂停中；2执行中；4已完成；100已删除',
  `job_remark` varchar(200) DEFAULT NULL COMMENT '备注',
  `uid` int(11) DEFAULT NULL COMMENT '用户ID',
  `user_name` varchar(45) DEFAULT NULL COMMENT '创建人姓名',
  `project_id` int(11) DEFAULT NULL COMMENT '项目组ID',
  `suite_id` int(11) DEFAULT 0 COMMENT '应用集ID',
  `app_id` int(11) DEFAULT NULL COMMENT 'appID',
  `app_name` varchar(45) DEFAULT NULL COMMENT '应用名称',
  `app_version` varchar(45) DEFAULT NULL COMMENT 'app版本',
  `pkg_id` int(11) DEFAULT NULL COMMENT '包ID',
  `package_name` varchar(128) DEFAULT NULL,
  `biz_code` int(11) DEFAULT NULL COMMENT '业务类型',
  `syspf_id` int(8) DEFAULT NULL COMMENT '系统平台',
  `task_content` mediumtext COMMENT '测试报文(JSON)',
  `task_desc` varchar(700) DEFAULT NULL COMMENT '任务描述',
  `create_time` bigint(20) DEFAULT NULL COMMENT '创建时间',
  `update_time` bigint(20) DEFAULT NULL COMMENT '更新时间',
  `channel_id` varchar(100) DEFAULT NULL COMMENT '应用渠道号',
  `ent_id` int(11) DEFAULT NULL COMMENT '企业id',
  `job_type` int(11) COMMENT '任务类型：0指定时间、1定点执行、2cron、3高级',
  `custom_job_rule` varchar(1000) DEFAULT NULL COMMENT 'job_type=3时的任务规则',
  `new_task_content` longtext COMMENT '新版测试报文',
  PRIMARY KEY (`job_id`),
  UNIQUE KEY `job_id_UNIQUE`(`job_id`),
  UNIQUE KEY `job_name`(`job_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=Compact;
```

## app处理服务 中的使用

### DAO/Service
- DAO: `QuartzJobInfoDAOImpl` -> `IQuartzJobInfoDAO`
- Mapper: `QuartzJobInfoMapper.xml`
- Service: `ScheduledJob.maintain/get/list`, `TemplateService`, `Quartz.appNameList/listQuartzJobByNames`, `TaskController.getTaskInfoByJobId`

### 关键SQL

| 操作 | SQL | 说明 |
| --- | --- | --- |
| 新增任务 | INSERT INTO quartz_job_info(job_name, job_rule, ...) VALUES (...) | 创建定时任务 |
| 条件查询 | SELECT ... FROM quartz_job_info WHERE ... ORDER BY create_time DESC | 模板/任务列表分页 |
| 批状态更新 | UPDATE quartz_job_info SET job_status=N WHERE job_id IN (...) | 批量暂停/恢复 |
| 转模板 | UPDATE quartz_job_info SET job_rule=NULL, job_type=N WHERE job_id=? | 清除调度规则 |
| 模板ID查询 | SELECT job_id FROM quartz_job_info WHERE project_id=? AND biz_code=? AND job_status IN(0,1,2,4) | 获取项目可用模板 |
