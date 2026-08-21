# view_suite_app

## 基本信息

| 属性 | 值 |
|------|-----|
| 数据库 | db_file |
| 业务域 | 套件管理 |
| 关联 Mapper | ViewSuiteAppMapper |
| 对象类型 | 视图（VIEW），非物理表 |

## 字段定义

| 字段名 | 类型 | 键 | 说明 |
|--------|------|-----|------|
| suite_id | Integer | — | 套件ID |
| suite_name | String | | 套件名称 |
| eid | Integer | | 企业ID |
| projectid | Integer | | 项目ID |
| pkgid | Integer | | 安装包ID |
| package_name | String | | 包名 |
| status | Integer | | 状态 |
| createtime | Long | | 创建时间 |
| updatetime | Long | | 更新时间 |

## 关联关系

- 视图源表：[suite_info](suite_info.md)（suite_id/suite_name/eid/projectid）与 [suite_app](suite_app.md)（pkgid/package_name）的联合视图
- 间接关联 [package_file](package_file.md)（pkgid）

## 相关接口

- [Suite](../../脚本服务/07-开放接口文档/套件管理/Suite.md)
- [SuiteController](../../脚本服务/07-开放接口文档/套件管理/SuiteController.md)

## 备注

- 只读视图，代码中仅有 `matchSuiteApp` 一个 SELECT（按 suiteId/pkgid/eid/projectid/packageName + status 精确匹配，LIMIT 1），用于校验套件与 App 包的匹配关系。
