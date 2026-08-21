---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# parameter_config_code

## 用途

监控参数配置码表。ParameterConfigCodeMapper/MyBatis 操作。AlarmLogServiceScheduledNew 读取动态阈值配置。存储温度/湿度等监控指标的报警阈值。

## 所属数据库

db_device_monitor

## DDL

```sql
CREATE TABLE `parameter_config_code`  (
                                          `id` int(11) NOT NULL AUTO_INCREMENT,
                                          `parameter_code` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '参数代码',
                                          `parameter_name` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '参数名称',
                                          `code_value` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '参数值',
                                          `code_name` varchar(40) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '参数说明',
                                          `create_time` datetime(0) NULL DEFAULT NULL COMMENT '创建时间',
                                          `update_time` datetime(0) NULL DEFAULT NULL COMMENT '修改时间',
                                          `create_user` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '创建人',
                                          `update_user` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '修改人',
                                          PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;
```

> DDL 来源：pocinit/src/mysql/db_device_monitor.sql（命中）

## 设备控制中心 中的使用

监控参数配置码表。ParameterConfigCodeMapper/MyBatis 操作。AlarmLogServiceScheduledNew 读取动态阈值配置。存储温度/湿度等监控指标的报警阈值。
