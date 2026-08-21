# service-SmsTaskApi — 短信发送代理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/notice/SmsTaskApi.java`（继承 `AbstractApi`，Spring bean 名 `notice.SmsTaskApi`）
> 类型：远端代理（→ NoticeManager 服务）
> 转发方式：V1 ApiServlet 前缀 `NoticeManager`，`action=sms`

## 方法列表

### 1. add — 创建短信发送任务

```java
public Integer add(Integer eid, Integer templetId, String tradeNo, String _to, JSONObject contentJson)
```

**用途**：向 NoticeManager 提交短信任务（sms_task），按模板渲染 content 后发送。

**转发目标**：`op=SmsTask.add`；data 含 `eid/templetId/tradeNo/to/content`。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业 id，null 返回 0 |
| templetId | Integer | 是 | 短信模板 id，null 返回 0 |
| tradeNo | String | 是 | 交易号，空返回 0 |
| _to | String | 是 | 接收手机号，空返回 0 |
| contentJson | JSONObject | 否 | 短信内容 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Integer | 短信任务 id（`result`，可为 null） |

**说明**：`eid/templetId/tradeNo/_to` 任一非法时记日志并返回 0，不发请求。

**调用者**：`NoticeServiceImpl.java`（`smstaskapi.add(eid, templetId, tradeNo, _to, msgData)`，任务完成短信通知）。

## 相关文档

- [00-分支索引](00-分支索引.md)
- NoticeManager
- [service-NoticeApi](service-NoticeApi.md) / [service-EmailTempletApi](service-EmailTempletApi.md)
