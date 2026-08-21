---
branch: syy.release.z7.8.1.0
module: filesystem
type: 产品功能
---

# App 与应用包管理

## 功能是什么

脚本服务承担 **App 包（APK/IPA/HAP）的查询与管理**：按包 ID / MD5 查包、按项目/套件分页列包、取下载 URL、补签名、移除。包本身的**上传与解析在文件管理服务（fileupload）完成**（写共享库 `db_file`），脚本服务是下游"提测选包"的数据来源。端到端视角见 [端到端-App包上传与使用](../../跨模块链路/端到端-App包上传与使用.md)。

## 核心能力

| 能力 | 说明 | 入口 |
|---|---|---|
| 条件分页列包 | 按 appId/版本/渠道/osType/projectId/包名/userIds/suiteId/packageId/ids 过滤 | `AppPackage.listPackageFile` |
| 列版本 | 按 appId + projectId 列版本（可选 suiteId） | `AppPackage.listAppVersion` |
| 查单包详情 | 按 packageId 或 appMd5 查包 + 应用 + 签名 + 文件 URL/MD5 | `AppPackage.getPackageFile` |
| 按 MD5 取下载 URL | 一条 SQL 直出 App 下载地址 | `AppPackage.getPackageFileByAppMd5` |
| 补签名信息 | 补全 `common_app` 签名字段 | `AppPackage.completeSingInfo` |
| 按项目列 App | 项目下的应用列表 | `AppPackage.listAppByProjectid` |
| 包扩展信息 | 对齐旧 ICE 查询（`PackageExt`） | `AppPackage.get` |
| 移除包 | 按 pkgid + suiteId 移除 | `AppPackage.remove` |

App 基本信息与子信息另有 `App` / `AppSubInfo`，控件查询 `Appcontrol`。

## 缓存与一致性

查询性能靠两级：JVM 内 Guava 本地缓存（`CommonFileModelCacheProvider` / `CommonAppModelCacheProvider`，写后 2 分钟过期）+ 跨服务显式清缓存 `FileApi.cleanCacheByFileId`。文件管理服务上传/更新包后会远程调本服务清 `common_file` 缓存。实现细节见 [AppPackage包查询实现](../03-实现逻辑/AppPackage包查询实现.md)。

## 入口文档

- [AppPackage](../07-开放接口文档/App与控件/AppPackage.md)（V1，`action=app`）
- [App](../07-开放接口文档/App与控件/App.md) / [AppSubInfo](../07-开放接口文档/App与控件/AppSubInfo.md) / [Appcontrol](../07-开放接口文档/App与控件/Appcontrol.md)

## 延伸阅读

- [AppPackage 包查询实现](../03-实现逻辑/AppPackage包查询实现.md)
- [端到端-App包上传与使用](../../跨模块链路/端到端-App包上传与使用.md)
