# service-ChannelEventV3Api — 通知事件配置查询代理（V3）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/notice/channel/ChannelEventV3Api.java`（@Component，继承 common 包 `AbstractApi`）
> 类型：远端代理（→ RealLogfile 服务）
> 转发方式：V3 REST，经 `ServiceRemoteV3Api.remoteGet`；域名 `ApiUtil.getServiceApiUrl("RealLogfile")`

## 方法列表

### 1. getNoticeEvent — 查询项目通知事件配置

```java
public List<NoticeEventResponseDTO> getNoticeEvent(Integer projectId, Integer eventType)
```

**用途**：查询项目在某事件类型（如 `NoticeEventEnum.TASK_FINISH` 任务完成）下配置的通知事件列表，定时任务结束后据此决定是否发机器人/邮件通知。

**转发目标**：

```java
// GET {RealLogfile}/v3/notice/notice/list?project_id={projectId}&event_type={eventType}
remoteV3Api.remoteGet(serviceApiUrl + "/v3/notice/notice/list?...", typeReference);
```

**说明**：整体 try/catch，异常时记日志并返回空列表（不向上抛）。

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectId | Integer | 否 | 项目 id（直接拼接到 query，无 null 校验） |
| eventType | Integer | 否 | 事件类型（如 `NoticeEventEnum.TASK_FINISH`） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | NoticeEventResponseDTO | 通知事件配置对象（字段见 RealLogfile 服务通知事件，代码未确认） |

**调用者**：
- `WebQuartz.java / 643` — Web 定时任务完成通知
- `McPcQuartz.java / 698` — PC 定时任务完成通知

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealLogfile](../../../平台基础功能服务/00-首页.md)
- [service-NoticeApi](service-NoticeApi.md)（实际发送通知）
- [WebQuartz](WebQuartz.md) / [McPcQuartz](McPcQuartz.md)
