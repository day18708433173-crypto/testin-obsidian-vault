---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# error_cause_log

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

问题原因日志表：记录按企业/项目上报的报错原因日志（含用例、问题单关联）。

## DDL

> 来源：pocinit/src/mysql（db_realcfg.sql / db_mcfg.sql / db_service.sql / db_user.sql）

```sql
CREATE TABLE `error_cause_log`  (
                                    `id` int(11) NOT NULL AUTO_INCREMENT,
                                    `error_cause_type_id` int(11) DEFAULT NULL COMMENT '问题类型id',
                                    `eid` int(11) NULL DEFAULT NULL COMMENT '企业id',
                                    `project_id` int(11) NULL DEFAULT NULL COMMENT '项目id',
                                    `error_cause` varchar(60) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '错误原因描述',
                                    `type` tinyint(1) NULL DEFAULT NULL COMMENT '类型(1app,3web 5pc)',
                                    `create_user_id` int(11) NULL DEFAULT NULL COMMENT '创建人',
                                    `create_user_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '创建人名称',
                                    `create_time` bigint(20) NULL DEFAULT NULL COMMENT '创建时间',
                                    `is_delete` tinyint(1) NULL DEFAULT 0 COMMENT '逻辑删除(0正常 1逻辑删除)',
                                    PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 3 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Compact;
```

## 索引

- `PRIMARY KEY (`id`) USING BTREE`

## 被哪些接口/mapper 方法使用

- `ErrorCauseLogDoMapper`（MyBatis）：insertErrorCauseLogDo、selectByCondition、countByErrorCauseByErrorCauseTypeId、updateById、updateQuestionIdByProjectIdEid、selectDistinctCase、selectById、updateErrorCauseLogByErrorCauseTypeId、selectCountErrorCauseLog
- 经 `ErrorCauseLogService` 被接口 [ErrorCauseLogController](../../平台配置（real-cfg）/07-开放接口文档/基础设施与问题管理/ErrorCauseLogController.md) 使用
