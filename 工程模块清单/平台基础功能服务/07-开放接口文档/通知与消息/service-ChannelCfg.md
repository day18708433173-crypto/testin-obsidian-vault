# service-ChannelCfg — 渠道配置（旧版 ApiServlet）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/service/channel/ChannelCfg.java`
> 类：`cn.testin.service.channel.ChannelCfg extends GenericBaseService`（非 Spring MVC Controller）
> 入口方式：ApiServlet 网关按 `action` 定位（`channel.ChannelCfg`），`op` 反射调用方法名；`ApiRequest.reqjson` 传参；返回 JSON 字符串。
> - **action**: `channel`（对应包 `cn.testin.service.channel`）
> - **入口格式**：`{"op": "ChannelCfg.方法名", "action": "channel", "data": {...}}`
> 依赖：`IChannelCfgService`（继承自 `GenericBaseService`）
> 业务：旧版渠道配置（渠道类别、名称、描述、状态）的增删改查。与 MVC 层 [NoticeChannelCfgController](NoticeChannelCfgController.md) 和新版 `newChannel.Channel` 功能重叠，属于旧版接口保留。

## 方法列表总表

| # | op | 方法 | 说明 |
|---|---|---|---|
| 1 | ChannelCfg.list | list | 渠道配置分页列表（可按 type/channelName/descr/status 筛选） |
| 2 | ChannelCfg.add | add | 新增渠道配置 |
| 3 | ChannelCfg.maintain | maintain | 更新渠道配置 |
| 4 | ChannelCfg.del | del | 按 channelName 删除渠道配置 |

统一返回：JSON 字符串。公共分页：`page<1` 归 1，`pageSize` 超限取 `Config.MaxSize`。

---

## 1. op=ChannelCfg.list — 渠道配置分页列表

### 请求格式
{"op": "ChannelCfg.list", "action": "channel", "data": {"type": ..., "channelName": "...", "page": ..., "pageSize": ...}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| type | 否 | 渠道类别（int），传了参与筛选 |
| channelName | 否 | 渠道名称，模糊匹配 |
| descr | 否 | 描述（注意：代码中 conditionMap 的 descr 错取了 `getInt` 而非 `getString`，可能是 bug） |
| status | 否 | 状态，必须为数字且 >= 0 |
| page / pageSize | 否 | 分页 |

### 响应结构

`data` 由 `baseListToResData` 从 `BaseList<DbChannelCfg>` 转换的分页结构。

### 调用链

```
ChannelCfg.list → IChannelCfgService.list(conditionMap, page, pageSize)
```

### 疑点

- descr 筛选字段用了 `reqjson.getInt("descr")` 而非 `getString`，按描述文本筛选将失败。

---

## 2. op=ChannelCfg.add — 新增渠道配置

### 请求格式
{"op": "ChannelCfg.add", "action": "channel", "data": {"channelName": "...", "type": ..., "descr": "...", "status": ...}}

### 请求参数（reqjson）

`DbChannelCfg.toBean(reqjson)` 反序列化，经 `validate` 校验：

| 字段 | 必填 | 说明 |
|---|---|---|
| channelName | 是 | 渠道名称，不能为空 |
| type | 是 | 渠道类别，>= 0 |
| descr | 是 | 渠道描述，不能为空 |
| status | 是 | 状态，>= 0 |
| createtime | 否 | 创建时间，非法时自动设为当前时间 |
| updatetime | 否 | 更新时间，非法时自动设为当前时间 |

### 响应结构

`data.result` = insert 条数。

### 调用链

```
ChannelCfg.add → validate(channelCfg) → IChannelCfgService.add(channelCfg)
```

---

## 3. op=ChannelCfg.maintain — 更新渠道配置

### 请求格式
{"op": "ChannelCfg.maintain", "action": "channel", "data": {"id": ..., "channelName": "...", ...}}

### 请求参数（reqjson）

与 add 相同，`DbChannelCfg.toBean` + `validate`，但走 `IChannelCfgService.update`。

### 响应结构

`data.result` = update 影响行数。

### 调用链

```
ChannelCfg.maintain → validate → IChannelCfgService.update(channelCfg)
```

---

## 4. op=ChannelCfg.del — 删除渠道配置

### 请求格式
{"op": "ChannelCfg.del", "action": "channel", "data": {"channelName": "..."}}

### 请求参数（reqjson）

| 字段 | 必填 | 说明 |
|---|---|---|
| channelName | 是 | 渠道名称（按名称删除，不是按 id） |

### 响应结构

`data.result` = 删除影响行数。

### 调用链

```
ChannelCfg.del → IChannelCfgService.delete(channelName)
```

---

## 备注

- `validate` 方法对 `createtime`/`updatetime` 做安全赋值：非法时 `setCreatetime/Updatetime(System.currentTimeMillis())`，无条件自动修正。
- descr 筛选 bug：`conditionMap.put("descr", reqjson.getInt("descr"))` 应为 `reqjson.getString("descr")`。

---

相关文档：[00-分支索引](00-分支索引.md) · [service-Channel](service-Channel.md)
