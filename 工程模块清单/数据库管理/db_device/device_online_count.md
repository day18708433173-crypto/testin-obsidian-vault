---
branch: syy.release.z7.8.1.0
module: real-controlcenter
type: SQL表
---

# device_online_count

## 用途

设备在线数统计表。DeviceOnlineCountService @Scheduled 每天凌晨1点统计各机型在线设备数写入。DeviceOnlineCountMapper/MyBatis 操作。

## 所属数据库

db_device

## DDL

> 未在 pocinit/src/mysql 中找到 DDL，以下为推断结构。数据库：db_device。

## 设备控制中心 中的使用

设备在线数统计表。DeviceOnlineCountService @Scheduled 每天凌晨1点统计各机型在线设备数写入。DeviceOnlineCountMapper/MyBatis 操作。
