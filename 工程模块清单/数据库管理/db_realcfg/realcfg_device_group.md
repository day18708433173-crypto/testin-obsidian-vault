---
branch: syy.release.z7.8.1.0
module: real-cfg
type: SQL表
---

# realcfg_device_group

- 数据库：`db_realcfg`
- 对象类型：表

## 用途

常用设备组表：以 eid+projectid+group_name 为业务主键保存用户常用设备集合。

## DDL

> pocinit DDL 来源未命中，以下字段根据 pojo + DAO SQL **推断**。

| 字段 | 类型（推断） | 说明 |
|---|---|---|
| eid | int | 企业 ID |
| projectid | int | 项目组 ID |
| userid | int | 操作人 ID |
| group_name | varchar | 设备组名称 |
| deviceids | varchar/text | 设备 ID 集合（逗号分隔） |
| create_time | bigint | 创建时间 |
| update_time | bigint | 更新时间 |

## 索引

- 推断：业务查询按 eid+projectid(+group_name) 过滤，主键/唯一键情况未知

## 被哪些接口/mapper 方法使用

- `DeviceGroupDAOImpl`（JDBC）← `DeviceGroupServiceImpl` ← 接口 [DeviceGroup](DeviceGroup.md)（addDeviceGroup/queryDeviceGroup/deleteDeviceGroup）
