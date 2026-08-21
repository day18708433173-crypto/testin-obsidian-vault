# service-CallbackApi — 任务完成 HTTP 回调

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/callback/CallbackApi.java`（SpringHelper Bean `callback.CallbackApi`）
> 类型：远端代理（→ NoticeManager 服务）
> 转发方式：V1 ApiServlet 前缀（`ApiUtil.doPress("NoticeManager", reqJson)`，action/op 反射路由）

## 方法列表

### 1. taskfinish — 任务完成回调

```java
public Integer taskfinish(Integer eid, String callbackUrl, String tradeNo, JSONObject contentJson)
```

**用途**：任务执行结束后，通过 NoticeManager 向客户侧 callbackUrl 发起 HTTP 回调通知（生成一条 HttpTask）。

**转发目标**：`action=http, op=HttpTask.add`（NoticeManager）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业 id，null 抛 `paraInvalid` |
| callbackUrl | String | 是 | 回调地址，空抛 `paraInvalid` |
| tradeNo | String | 是 | 外部交易号/关联单号，空抛 `paraInvalid` |
| contentJson | JSONObject | 是 | 回调内容，null 抛 `paraInvalid` |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Integer | HttpTask id（可为 null） |

**调用者**：
- `NoticeServiceImpl.java`（`callbackApi.taskfinish(eid, url, tradeNo, resultJson)`）

## 相关文档

- [00-分支索引](00-分支索引.md)
- NoticeManager
- [service-NoticeReportApi](service-NoticeReportApi.md)
