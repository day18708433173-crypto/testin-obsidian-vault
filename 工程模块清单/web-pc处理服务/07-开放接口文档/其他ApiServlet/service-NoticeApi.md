# service-NoticeApi — 邮件/消息通知与渠道查询代理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/notice/NoticeApi.java`（继承 `AbstractApi`）
> 类型：远端代理（→ NoticeManager 服务）
> 转发方式：V1 ApiServlet 前缀 `NoticeManager`

## 方法列表

### 1. addEmailTask — 发送邮件

```java
public boolean addEmailTask(EmailTask task) throws GeneralException
```

**转发目标**：`action=email, op=EmailTask.add`。

**请求参数**（`EmailTask`）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| tradeNo | String | 是 | 交易号/关联单号，空抛 `paraInvalid` |
| type | Integer | 是 | 邮件类型，限 0/1（null 或越界抛 `paraInvalid`） |
| subject | String | 是 | 主题，空抛 `paraInvalid` |
| content | String | 是 | 内容，空抛 `paraInvalid` |
| to | String | 是 | 收件人，空抛 `paraInvalid` |
| level | Integer | 否 | 优先级，<0 抛 `paraInvalid` |
| templetId | Integer | 否 | 模板 id |
| cc | String | 否 | 抄送 |
| bcc | String | 否 | 密送 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Boolean | 发送成功（远端 result > 0）为 true，否则 false |

**调用者**：`NoticeServiceImpl.java` — 任务完成邮件通知。

### 2. addMsgTask — 发送新渠道消息（钉钉、飞书等）

```java
public boolean addMsgTask(MsgTask task) throws GeneralException
```

**转发目标**：`action=newChannel, op=MsgTask.add`。

**请求参数**（`MsgTask`）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| tradeNo | String | 是 | 交易号，空抛 `paraInvalid` |
| subject | String | 是 | 主题，空抛 `paraInvalid` |
| content | String | 是 | 内容，空抛 `paraInvalid` |
| to | String | 是 | 接收人，空抛 `paraInvalid` |
| eid | Integer | 否 | 企业 id |
| projectid | Integer | 否 | 项目 id |
| channelId | String | 否 | 渠道 id |
| eventType | Integer | 否 | 事件类型 |
| templetId | Integer | 否 | 模板 id |
| level | Integer | 否 | 优先级，<0 抛 `paraInvalid` |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| result | Boolean | 发送成功（远端 result > 0）为 true，否则 false |

**调用者**：`NoticeServiceImpl.java`。

### 3. channelList — 查询通知渠道列表

```java
public List<NoticeChannelCfg> channelList(Integer eid, Integer projectId) throws GeneralException
```

**转发目标**：`action=newChannel, op=Channel.list`，data 含 `eid/projectId`；结果从 `result` 字段反序列化。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 否 | 企业 id |
| projectId | Integer | 否 | 项目 id |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | NoticeChannelCfg | 通知渠道配置对象（字段见 NoticeManager 服务 `Channel.list`，代码未确认） |

**调用者**：`NoticeServiceImpl.java` — 发送前取项目配置的渠道（robotNotice 场景）。

## 相关文档

- [00-分支索引](00-分支索引.md)
- NoticeManager
- [service-EmailTempletApi](service-EmailTempletApi.md) / [service-SmsTaskApi](service-SmsTaskApi.md)
