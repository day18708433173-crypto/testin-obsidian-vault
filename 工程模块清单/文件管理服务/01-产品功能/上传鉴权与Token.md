---
branch: syy.release.z7.8.1.0
module: fileupload
type: 产品功能
---

# 上传鉴权与 Token

> 上传文件前，平台要确认"谁在上传、有没有资格上传"。文件管理服务的鉴权按调用方分成三类，彼此独立，用哪个取决于是谁在调：页面/平台用户走 **sid 登录态**，第三方业务服务走 **token 开放凭证**，内部运维工具走 **硬编码口令**。代码级细节见 [横切-上传鉴权体系](../04-复杂功能细节/横切-上传鉴权体系.md)。

## 三类鉴权一句话定位

| 鉴权方式 | 面向谁 | 凭证形态 | 校验依据 |
|---|---|---|---|
| sid 登录态 | 平台用户 / 页面端上传 | 会话标识 sid | 拿 sid 到平台基础功能服务换 `UserOnline`（userid/eid） |
| token 开放凭证 | 第三方业务服务（自动打包流水线等） | `serviceId` + `accessToken` | 先查服务注册表、再比 Redis 里的 token 值 |
| 硬编码口令 | 内部运维（脚本批量重检） | 固定 key `fa568093e6354882a534dbce946c9d53` | 代码内写死比对 |

## sid 登录态鉴权

上传 App、分片、脚本步骤重建等入口都会拿 `sid` 去平台基础功能服务问"这个人是谁"，核心动作是 `AuthApi.getUserOnline(sid)`（向 `UserManager` 前缀发起 `Online.getUserOnline`），拿到 `UserOnline{userid, eid, uname}` 再继续。各入口的严格程度不同：

- `/file/upload-app`（`FileUploadController.uploadApp`）：配置 `public_access_auth` 默认 true 时强制校验 sid，false 时跳过（留给内部调用）；
- `/file_system/app/upload`（`AppV3Controller.uploadApp`）：强制校验，失败返回 "sid is invalid"；
- 分片上传（`AppUploadController.checkAuth`）：除了 sid，还要 `AuthApi.myProjects` 校验"该用户属于这个项目组"，并确认 suiteId 有效；
- 脚本步骤重建（`RebuildController.scriptStep`）：sid 认证失败即返回 "sid认证失败"。

## token 开放鉴权

这是为第三方业务服务（如自动打包流水线）准备的**服务级**凭证，有"注册 → 签发 → 校验"三段：

1. **注册**：`POST /token/registerService` 传 `serviceName`，平台生成私钥 `privateKey` 写入 `upload_service_auth` 表并返回给调用方；
2. **签发**：`GET /token/get` 传 `serviceId` + `publicKey`（`publicKey` 必须等于 `MD5(serviceId + privateKey)`），校验通过后生成一个 UUID token 写入 Redis `fileUpload_{serviceId}`，有效期 7200 秒（2 小时）；
3. **校验**：上传时带 `serviceId` + `accessToken`，依次确认 serviceId 已注册、Redis 里有这个 token、值能对上。

**需要注意**：token 的"签发接口 + 校验代码"在本分支都是齐全的，但校验逻辑入口（`parseHttpRequestWithToken`）当前**没有任何上传端点真正调用**——也就是说第三方上传目前并没有真正被 token 保护，这套机制处于"定义完备、未接线"状态。不要误以为 token 已生效。

## 内部硬编码口令

脚本校验相关的内部接口共用一个硬编码 key `fa568093e6354882a534dbce946c9d53`（`ApiController.checkScript` 系列 + 旧体系 `fs.Script.checkScript`）。它是内部约定口令，不应出现在公开文档或前端。

## 附：跨服务调用方向（以代码为准）

鉴权换取登录态、绑定应用、通知结果时，本服务会向其它服务发起调用。方向一律以代码里的 `ApiUtil.doPress(prefix, op)` 为准：

| 动作 | 目标服务前缀 | 说明 |
|---|---|---|
| 换登录态 / 查项目组 / 查用户 | `UserManager` | 平台基础功能服务 |
| 清脚本缓存 / 取最新脚本 | `Script` | 脚本服务 filesystem（注意 `BaseApi.fileManagement` 实际值是 `"Script"`，不是 `"File"`） |
| 绑定 App 与应用 | `Script` | 复用 `fileManagement`，op=Suite.bind |
| 任务完成通知 | `RealTest` | app处理服务 real-test |

## 延伸阅读

- [横切-上传鉴权体系](../04-复杂功能细节/横切-上传鉴权体系.md) — 三类鉴权的代码位置与校验步骤
- [文件上传与存储实现](../03-实现逻辑/文件上传与存储实现.md) — token 签发/校验的完整代码链路
- [横切-文件上传责任链](../04-复杂功能细节/横切-文件上传责任链.md) — 鉴权通过后进入的处理链
