# app_sub_info

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | App 管理 |
| 关联 Mapper | AppSuInfoMapper |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| app_name | — | — | 应用名称 |
| eid | — | — | 企业ID |
| app_sub_name | — | | 子应用名称 |
| descr | — | | 描述 |
| third_user_id | — | | 第三方用户ID |
| status | — | | 状态 |
| createtime | Long | | 创建时间 |
| updatetime | Long | | 更新时间 |
| content | — | | 内容 |
| app_id | — | | 应用ID |

## 关联关系

- 引用：[common_app](common_app.md)（app_id FK）

## 相关接口

- [AppSubInfoController](AppSubInfoController.md)
