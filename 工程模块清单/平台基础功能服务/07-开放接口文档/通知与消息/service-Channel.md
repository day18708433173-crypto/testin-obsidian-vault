# service-Channel — 消息通道（新版）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/newChannel/Channel.java`
> 类：`cn.testin.service.newChannel.Channel extends GenericBaseService`（非 Spring MVC Controller）
> 入口方式：ApiServlet 网关按 `action` 定位（`newChannel.Channel`），`op` 反射调用方法名；`ApiRequest.reqjson` 传参；返回 JSON 字符串。
> - **action**: `newChannel`（对应包 `cn.testin.service.newChannel`）
> - **入口格式**：`{"op": "Channel.方法名", "action": "newChannel", "data": {...}}`
> 依赖：`IMsgChannelCfgService`（继承自 `GenericBaseService`）
> 业务：新版消息通道管理——按企业（eid）+ 项目组（projectId）维度管理消息通道配置，支持列表、新增、更新、批量删除、当前企业可用通道类型查询。

## 方法列表总表

| # | op | 方法 | 说明 |
|---|---|---|---|
| 1 | Channel.list | list | 按企业/项目组查询可用消息通道列表 |
| 2 | Channel.add | add | 新增消息通道配置 |
| 3 | Channel.update | update | 更新消息通道配置 |
| 4 | Channel.del | del | 批量删除消息通道（按 ids 数组） |
| 5 | Channel.channelList | channelList | 查询该企业下可用的通道类型列表 |

统一返回：JSON 字符串。注意：`list` 不分页（全量返回），其他操作传 eid 标识企业。

---

## 1. op=Channel.list — 消息通道列表

### 请求格式
{"op": "Channel.list", "action": "newChannel", "data": {"eid": ..., "projectId": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 否 | 企业 ID，传了校验 > 0 |
| projectId | 否 | 项目组 ID，传了且 > 0 才参与筛选 |

### 响应结构

`data.result` = `JSONArray`（`List<NoticeChannelCfg>` 序列化），全量返回，不分页。

`NoticeChannelCfg` 元素字段：

| 字段 | 类型 | 说明 |
|---|---|---|
| id | Integer | 通道主键 |
| eid | Integer | 企业 ID |
| projectid | Integer | 项目组 ID |
| projectName | String | 项目名称（非表字段，查询时填充） |
| type | Integer | 通道类型（2 钉钉 webHook / 3 joyChat） |
| channelName | String | 通道名称 |
| descr | String | 描述 |
| dependentInfo | String | 依赖信息 |
| defaultSelect | Integer | 是否默认选中 |
| defaultTempletId | Integer | 默认模板 ID |
| status | Integer | 状态 |
| createtime | Long | 创建时间 |
| updatetime | Long | 更新时间 |
| config | String | 配置信息（JSON 字符串，内含 webHookUrl/secret/passRateConfig） |

### 实现意图

按 eid（必选逻辑？代码中 eid 不传时不报错但筛选可能全量）和可选 projectId 查询通道列表。与旧版 `ChannelCfg.list` 不同：此处分页由全量列表替代。

### 调用链

```
Channel.list → IMsgChannelCfgService.list(eid, projectid)
```

---

## 2. op=Channel.add — 新增消息通道

### 请求格式
{"op": "Channel.add", "action": "newChannel", "data": {"eid": ..., "channelName": "...", "config": "..."}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| eid | Integer | 否 | 企业 ID，>= 0 放行，< 0 返回 paraInvalid |
| projectid | Integer | 否 | 项目组 ID，>= 0 放行 |
| type | Integer | 否 | 通道类型，>= 0 |
| channelName | String | 是 | 通道名称，不能为空 |
| descr | String | 否 | 描述 |
| config | String | 是 | 配置信息（JSON 字符串），不能为空 |

### 响应结构

`data.result` = insert 条数。

### 实现意图

新增一条消息通道配置，关键点是 `channelName` 和 `config` 必须非空，eid/projectid/type/descr 可选。时间戳自动设为当前。

### 调用链

```
Channel.add → IMsgChannelCfgService.add(noticeChannelCfg)
```

---

## 3. op=Channel.update — 更新消息通道

### 请求格式
{"op": "Channel.update", "action": "newChannel", "data": {"id": ..., "channelName": "...", "config": "..."}}

### 请求参数（reqjson）

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| id | Integer | 是 | 通道主键，>= 1 |
| eid | Integer | 否 | 企业 ID |
| projectid | Integer | 否 | 项目组 ID |
| type | Integer | 否 | 通道类型 |
| channelName | String | 是 | 通道名称 |
| descr | String | 否 | 描述 |
| config | String | 是 | 配置信息 |

### 响应结构

`data.result` = update 影响行数。

### 调用链

```
Channel.update → IMsgChannelCfgService.update(noticeChannelCfg)
```

---

## 4. op=Channel.del — 批量删除消息通道

### 请求格式
{"op": "Channel.del", "action": "newChannel", "data": {"eid": ..., "ids": [...]}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 是 | 企业 ID，>= 1 |
| ids | 是 | 通道主键数组（JSONArray），不能为空 |

### 响应结构

`data.result` = 删除影响行数。

### 实现意图

批量删除：`ids` 经 Gson 反序列化为 `List<Integer>`，连同 eid 传给 `IMsgChannelCfgService.delete(eid, null, ids)`（第二个参数可能为 projectId，此处传 null）。

### 调用链

```
Channel.del
├─ Config.gson.fromJson(ids, List<Integer>)
└─ IMsgChannelCfgService.delete(eid, null, ids)
```

---

## 5. op=Channel.channelList — 企业可用通道类型列表

### 请求格式
{"op": "Channel.channelList", "action": "newChannel", "data": {"eid": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| eid | 否 | 企业 ID，>= 1 |

### 响应结构

| 字段 | 类型 | 说明 |
|---|---|---|
| data.result | List\<Integer\> | 可用的通道类型编码列表 |
| data.list | List\<Map\<String, String\>\> | 通道类型编码对应的中文名称列表（由 `MsgTypeEnum.getInfoByCodes` 翻译） |

### 实现意图

查询某企业下已配置的通道类型编码集合，并翻译为中文名供前端展示。

### 调用链

```
Channel.channelList
├─ IMsgChannelCfgService.channelList(eid)
└─ MsgTypeEnum.getInfoByCodes(result)
```

---

## 疑点

- `list` 方法中 eid 校验 `reqjson.getInt("eid") < 1` 才报错，eid 不传 时 `reqjson.isNull` 不判 null 直接走 else 分支，可能取到默认值或 NPE。
- `add` 中 `projectid` 变量命名不统一（驼峰与下划线混用）。

---

相关文档：[00-分支索引](00-分支索引.md) · [NoticeChannelCfgController](NoticeChannelCfgController.md) · [service-ChannelCfg](service-ChannelCfg.md) · [service-MsgTask](service-MsgTask.md)
