---
tags: [常见问题]
---

# 前端联调FAQ

## 登录态与访问

### Q: 直接访问 dev server 根路径，跳到了 `/unauthorized`？

前端是 iframe 子系统，路由守卫拿不到 `sid` 就跳未授权页。开发时从主平台入口跳进，或手动在 URL 补 `?sid=...&projectId=...`。注意：URL 上带 `skey` 时守卫会跳过校验（那是分享报告免登场景，不是登录态）。见 [登录态与主平台集成](../03-实现逻辑/登录态与主平台集成.md)。

### Q: 请求里 `sid` / `projectId` / `userId` 要不要自己传？

**不要**。`src/utils/Http.ts` 请求拦截器会自动注入到 body/params，并加 `Sid` / `projectId` 请求头。手动传会造成重复或覆盖。见 [前端HTTP层与响应约定](../03-实现逻辑/前端HTTP层与响应约定.md)。

### Q: 联调时后端说没收到 sid？

检查两点：① 页面 URL/Session 里是否真有 sid（iframe 链路是否走通）；② 是否绕过了 `Http.ts` 直接 new 了 axios——全项目只允许用 `Http.ts` 那一个实例。

## 代理与路径

### Q: `pnpm dev` 请求 404，路径里带着 `/api/v1`？

后端控制器**没有 `/api/v1` 统一前缀**（生产环境由前置网关剥离）。把 proxy target 改成本地后端 `http://127.0.0.1:8526` 时，必须同时取消注释 `vite.config.ts` 里那行 `rewrite: path => path.replace(/^\/api\/v1/, '')`。

### Q: 切了 proxy target 后还是连到旧环境？

vite.config.ts 里有多行注释掉的备选 target，确认改的是生效那行、且没有两行 target 并存；改完重启 dev server。

## 响应与错误处理

### Q: 接口明明返回成功，前端却弹错？

本平台成功码是 **`code === 200`**——不要套用主平台 `code === 0` 的约定。`505` / `508` 会原样返回给调用方自行处理（如锁冲突、权限场景），其余非 200 统一 `ElMessage` 弹错并 throw。

### Q: 报告页刷新后数据全丢 / 报错？

报告页（`/report/1|2`、`/share/report/1|2`）设计为**新页签打开**，Pinia store 丢失时从 `sessionStorage` 的 `store` key 恢复（`Http.ts` 拦截器里处理）。如果你改了报告页入口却没有写 sessionStorage，刷新就会丢。

### Q: 路由跳转拼接的 URL 不生效？

路由用 **hash history**（`createWebHashHistory`），站内链接要 `#/` 开头。

## 权限

### Q: 新页面所有人都能访问，权限没拦住？

路由没配 `meta.authKey` 等于不做权限拦截。一级路由 `authKey` 必须是**数组**；key 要与 `src/directives/permission/const.ts` 及后端 configKey 对齐。见 [权限体系](../03-实现逻辑/权限体系.md)。

### Q: `v-auth` 指令加了但按钮还在/永远不显示？

值可以是单个 key 或数组（数组任一命中即显示）；无权限时节点被 `removeChild` 移除（不是 v-if，后续状态变化不会自动回来）。先确认 `usePermission` store 里权限串列表真的含该 key——`api-all` 才是全权限。

## 编辑锁

### Q: 保存/删除接口返回"资源被锁定"？

这是 [编辑锁机制](../03-实现逻辑/编辑锁机制.md) 的正常表现：他人正在编辑。前端对 rename/copy/delete/save 统一先调 `checkLockedByApi`。排查锁归属与过期看 [编辑锁机制](../03-实现逻辑/编辑锁机制.md) 和后端 `PageLockService`。

## 请求与刷新疑难（源码确认）

### Q: DELETE 请求偶发缺 `sid` / `projectId` / `userId`？

`http.delete` 的第二参数是 params（`Http.ts`）；调用方既没传 params 也没传 data 时，拦截器的注入分支（`method==='delete' && !config.data`）可能落空。DELETE 接口显式传空 data 或 params 占位。

### Q: 刷新/新开页签后 projectId 丢了？

按这个顺序排查（详见 [登录态与主平台集成](../03-实现逻辑/登录态与主平台集成.md)）：
1. `sessionStorage.store` 有没有值（`main.ts` 的订阅写入）；
2. 路由守卫的恢复逻辑（`router/index.ts`）；
3. 请求拦截器的兜底恢复（`src/utils/Http.ts`）。

### Q: sid 明明在 URL 里，后端却按无 projectId 处理？

`KeyInterceptor.saveProjectIdToSession` 解析 body JSON 失败时**只 log 不抛**（`KeyInterceptor.java`），projectId 会以 null 静默跳过。检查请求 body 是否为合法 JSON、是否被 wrapper 提前读过流（见 [后端请求处理管线](../03-实现逻辑/后端请求处理管线.md)）。

## 相关阅读

- [前端开发指南](../06-开发指南/前端开发指南.md) / [前端HTTP层与响应约定](../03-实现逻辑/前端HTTP层与响应约定.md) / [登录态与主平台集成](../03-实现逻辑/登录态与主平台集成.md) / [构建与部署FAQ](构建与部署FAQ.md)
