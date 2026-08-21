---
tags: [复杂功能细节]
---

# Mock服务

## 概述

平台为 HTTP / TCP 接口提供 Mock 能力：**接口定义里挂"期望"（匹配规则 + 返回内容）与 Mock 脚本，真实的 Mock 应答由一个独立部署的 mock 服务进程完成**——本仓库只负责 Mock 数据的编辑、存储和按接口吐出配置，应答流量不进本平台服务端。

- Mock 配置存 `test_interface_mock` 表（主键即 interfaceId），随接口走；
- 接口保存时自动生成 **Mock 地址**（前端展示用 / 集群内回调用各一套 URL）；
- 独立的 mock 服务（K8s 服务名 `api-test-mock`，HTTP 8010 / TCP 8011）通过本平台提供的两个**无鉴权内部接口**拉取期望与脚本，完成请求匹配与应答。

相关文档：[接口管理](../01-产品功能/接口管理.md)、[编辑锁机制](../03-实现逻辑/编辑锁机制.md)、[部署与打包](../02-技术架构/部署与打包.md)。

## 逻辑详解

### 1. 数据结构

`TestInterfaceMockDoWithBLOBs`（`src/main/java/cn/testin/dao/TestInterfaceMockDoWithBLOBs.java`）：`expect`（JSON 数组）、`script`（JSON 对象）、`delay`（延时秒）——一个接口一条记录。

- 期望 `MockExpect`（`entity/dto/MockExpect.java`）：`name`、`available`（启用开关）、`result`（返回内容）、`match`（匹配条件列表）；
- 匹配条件 `MockExpectMatch`（`entity/dto/MockExpectMatch.java`）：`pos`（参数位置：HTTP 为 `query/path/header/cookie/body`；TCP 固定用 `sendText`）、`key`（参数名）、`op`（操作符：eq/gt/lt 等，见前端 `utils/Utils.ts` 的 `options`）、`val`（参数值）；
- 脚本 `MockScript`（`entity/dto/MockScript.java`）：`content`（JavaScript）+ `available`，用于期望之外的自定义动态应答。

### 2. 编辑与保存（前端）

接口定义页 `ApiDefinition.vue`（`testin-api-frontend/src/views/api/components/ApiDefinition.vue`）下半区即 Mock 编辑区：期望列表（`MockExpectDefineDialog.vue`）、Mock 脚本（`MockScriptDefineDialog.vue`，JS 代码编辑器）、HTTP 协议下还有"延时响应(s)" 0-300 秒。`watch(mock)` + 300ms 防抖自动保存，保存接口为 `POST /interface/mock/save`（`api/interface.ts`）。

编辑交互细节：

- 打开接口时 `getInterfaceMockByApi` 拉取并反序列化 expect/script 回填（`api/interface.ts`）；没有任何 Mock 配置时后端返回空，前端用默认值兜底；
- 保存时 `expect`、`script` 两个字段分别 `JSON.stringify` 后随 `delay` 一起提交（`api/interface.ts`）——即库里两个 BLOB 列就是前端对象的序列化串；
- 期望弹层里 `pos`（参数位置）下拉仅 HTTP 协议显示（`MockExpectDefineDialog.vue` 的 `v-show`），操作符来自 `utils/Utils.ts` 的 `options`（eq/gt/lt 等）；
- 页面上的 `canEdit` 控制期望/脚本弹层的"确定"按钮禁用态，与接口编辑权限联动（`MockExpectDefineDialog.vue`）。

后端 `InterfaceMockService.save`（`src/main/java/cn/testin/service/InterfaceMockService.java`）：

- 先过**编辑锁** `checkLockedForApiDirectOperation`，别人在编辑该接口时直接拒绝，见 [编辑锁机制](../03-实现逻辑/编辑锁机制.md)；
- 按 interfaceId 判断 insert / update，并记录用户行为（`userBehaviorService.recordUserBehavior`，动作分别为 INSERT/UPDATE）。

### 3. Mock 数据结构在接口生命周期的归宿

- 接口查询/保存返回的 `mock` 字段就是前端展示的 mock 地址（见下节）；
- 接口被彻底删除（回收站清理）时，Mock 记录按主键一并删除——`RecycleBinService.java` 调 `testInterfaceMockDoMapper.deleteByPrimaryKey`；
- 接口进回收站但未清理时 Mock 记录保留，还原后可继续使用。

### 4. Mock 地址的生成与路由

接口保存/查询时由 `InterfaceService` 自动填充 mock 地址：

```java
// setBackEndMockAddress（InterfaceService.java）
HTTP: mock = mock.http.backend.url + projectId + path
      （非默认 mock 时追加 ?testinInterfaceId={id}）
TCP:  mock = mock.tcp.backend.url
```

- `setMockAddress`用 `mock.http.frontend.url` 拼**给用户看**的地址；`setBackEndMockAddress`用 `mock.http.backend.url` 拼**集群内使用**的地址；
- 生产配置（`application-prod.yml`）：frontend `http://api-test.${WEB_DOMAIN}/mock/`（经网关到 mock 服务），backend `http://api-test-mock:8010/`（K8s 集群内直连）；TCP 分别为 `api-test.${WEB_DOMAIN}` / `api-test-mock:8011`；
- **同 path 多接口**时只有一个"默认 Mock"（`isDefaultMock`，`InterfaceService.handleIsDefaultMock`），非默认接口的 Mock 地址带 `testinInterfaceId` 查询参数以便 mock 服务精确匹配。

Mock 地址形如（HTTP、默认 Mock）：

```
http://api-test.{域名}/mock/{projectId}{接口path}                     ← 用户侧
http://api-test-mock:8010/{projectId}{接口path}                       ← 集群内
http://api-test-mock:8010/{projectId}{接口path}?testinInterfaceId={id} ← 非默认 Mock
```

保存请求的 DTO 是 `TestInterfaceMockSaveDTO`（`entity/dto/parse/TestInterfaceMockSaveDTO.java`）：interfaceId + userId + expect/script 两个已序列化的 JSON 串 + delay；`save` 不校验内容合法性，原样入库。

### 5. Mock 服务如何拿到配置（平台 → mock 进程）

`InterfaceMockController`（`src/main/java/cn/testin/controller/InterfaceMockController.java`）提供 4 个端点：

| 端点 | 鉴权 | 消费方 |
|---|---|---|
| `POST /interface/mock/save` | `api-interface-save` | 前端编辑 |
| `GET /interface/mock/view/{interfaceId}` | `api-interface-view` | 前端回显 |
| `GET /interface/mock/viewJson/{interfaceId}` | **无** | mock 服务拉单接口配置 |
| `GET /interface/mock/TCP/list` | **无** | mock 服务拉全部 TCP 期望 |

- `viewJson` → `getMockDTO`（`InterfaceMockService.java`）：过滤出 `available=true` 的期望，脚本仅在启用时返回内容，附带 `delay`；输出对象是精简的 `MockDTO`（`entity/dto/MockDTO.java`：expect 列表 + script 内容 + delay），期望项被裁成 `MockExpectDTO`（name/match/result，不含 available）；
- `TCP/list` → `listTcpMockExpectDTO`：扫全部 `protocol=TCP` 的接口，汇总其启用期望——TCP Mock 没有 URL 可路由，只能全量下发按 `sendText` 匹配。

```mermaid
flowchart LR
    U[用户/被测系统] -->|请求 mock 地址 /| GW[网关/直连]
    GW --> MS[api-test-mock 独立服务]
    MS -->|GET /interface/mock/viewJson/{id}| API[本平台服务端]
    MS -->|GET /interface/mock/TCP/list| API
    API --> MS
    MS -->|匹配成功返回 result，支持 delay| U
```

## 关键代码位置

| 功能 | 位置 |
|---|---|
| Mock REST 接口 | `src/main/java/cn/testin/controller/InterfaceMockController.java` |
| Mock 保存（含编辑锁） | `src/main/java/cn/testin/service/InterfaceMockService.java` |
| 单接口 Mock 输出（mock 服务用） | `src/main/java/cn/testin/service/InterfaceMockService.java` |
| 全部 TCP 期望输出 | `src/main/java/cn/testin/service/InterfaceMockService.java` |
| Mock 地址拼接 | `src/main/java/cn/testin/service/InterfaceService.java`、 |
| 默认 Mock 判定 | `src/main/java/cn/testin/service/InterfaceService.java` |
| 回收站连带删除 Mock | `src/main/java/cn/testin/service/RecycleBinService.java` |
| Mock 地址配置项 | `src/main/resources/application-prod.yml` |
| 前端 Mock 编辑区 | `testin-api-frontend/src/views/api/components/ApiDefinition.vue` |
| 期望定义弹层 | `testin-api-frontend/src/components/MockExpect/MockExpectDefineDialog.vue` |
| 脚本定义弹层 | `testin-api-frontend/src/components/MockExpect/MockScriptDefineDialog.vue` |

## 注意事项与坑

1. **Mock 应答进程不在本仓库**：匹配引擎、脚本沙箱、`/mock/**` 流量入口都属于独立的 `api-test-mock` 部署单元（HTTP 8010、TCP 8011）。本仓库改动 expect/script 的数据结构时，必须同步确认 mock 服务的解析逻辑，见 [部署与打包](../02-技术架构/部署与打包.md)。
2. `viewJson` 与 `TCP/list` **没有 `@ApiConfigKey` 鉴权**，安全完全依赖网络隔离；不要把这两个端点暴露到网关外网。
3. 匹配条件里 TCP 协议的参数名目前只支持 `sendText`（`MockExpectDefineDialog.vue` 的 placeholder 有明示），别给 TCP 期望配 header/query 类条件。
4. `delay` 仅 HTTP 协议在界面上暴露（`v-show`），单位是秒、上限 300；TCP 期望即使库里有 delay 也不会被界面编辑。
5. 期望匹配是"先过滤 available 再逐条匹配"（`getMockDTO`），停用期望不参与应答；**全部期望停用时 mock 服务拿到空列表**，行为取决于 mock 服务自身兜底（通常是 404）。
6. 同 path 建第二个接口时它会自动变成非默认 mock（`isDefaultMock=0`），其 mock 地址必须带 `testinInterfaceId` 才能命中——复制接口后直接用原 mock 地址会打到默认接口的期望上。
7. 保存走编辑锁直查（`checkLockedForApiDirectOperation`），前端另有 `checkLockedByApi` 预检；绕过前端直接调 save 接口依然会被锁拦截。
8. 前端对 mock 的保存是 watch 防抖自动触发，进入接口页即触发一次 `getInterfaceMockByApi` 回显；接口被删后 mock 记录由回收站逻辑连带处理，见 [回收站](../01-产品功能/回收站.md)。
