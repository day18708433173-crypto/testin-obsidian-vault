---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# cabinet_space_cfg

## 用途

机柜空间配置表。记录机柜层数、行列信息。

## 所属数据库

db_device

## DDL

```sql
CREATE TABLE `cabinet_space_cfg`  (
                                      `case_floor` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '机柜层数',
                                      `case_row` int(11) NULL DEFAULT NULL COMMENT '行数',
                                      `case_col` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '列数',
                                      `case_num` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '机柜信息'
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '机柜空间配置信息' ROW_FORMAT = Compact;
```

> DDL 来源：pocinit/src/mysql/db_device.sql（命中）

## 设备控制中心 中的使用

机柜空间配置表。记录机柜层数、行列信息。
