# script_control

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 脚本控件 |
| 关联 Mapper | ScriptControlMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| id | — | PK | 主键ID |
| projectid | — | | 项目ID |
| appid | — | | 应用ID |
| userid | — | | 用户ID |
| name | — | | 控件名称 |
| type | — | | 控件类型 |
| fingerprint | — | | 控件指纹 |
| bigImage | — | | 大图 |
| smallImage | — | | 小图 |
| thumbImage | — | | 缩略图 |
| version | — | | 版本 |
| targetAppVer | — | | 目标应用版本 |
| descr | — | | 描述 |
| info | — | | 信息 |
| status | Short | | 状态 |
| createtime | Long | | 创建时间 |
| updatetime | Long | | 更新时间 |
| expr | — | | 表达式 |

## 关联关系

- 被引用：[script_control_sequence](script_control_sequence.md)（script_control_id FK）

## 相关接口

- [ScriptControlController](ScriptControlController.md)
