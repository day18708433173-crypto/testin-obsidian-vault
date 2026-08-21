# service-EmailTempletApi — 自定义邮件模板查询代理

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/notice/EmailTempletApi.java`（继承 `AbstractApi`）
> 类型：远端代理（→ NoticeManager 服务）
> 转发方式：V1 ApiServlet 前缀 `NoticeManager`，`action=email`

## 方法列表

### 1. getEmailTemplet — 按项目查自定义邮件模板

```java
public EmailTempletConfig getEmailTemplet(Integer projectId) throws GeneralException
```

**用途**：查询项目组配置的自定义邮件模板（email_templet），任务完成邮件发送前用于覆盖默认模板。

**转发目标**：

```java
reqJson.put("action", "email");
reqJson.put("op", "EmailTempletCfg.get");
// data: projectId
```

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectId | Integer | 是 | 项目 id，null 抛 `paraInvalid` |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | EmailTempletConfig | 自定义邮件模板对象（字段见 NoticeManager 服务 `EmailTempletCfg.get`，代码未确认） |

**调用者**：`NoticeServiceImpl.java`（`templetApi.getEmailTemplet(...)`，邮件组装流程）。

## 相关文档

- [00-分支索引](00-分支索引.md)
- NoticeManager
- [service-NoticeApi](service-NoticeApi.md)
