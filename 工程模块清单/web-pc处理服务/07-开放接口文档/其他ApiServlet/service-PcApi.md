# service-PcApi — 浏览器（PC）去重列表

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/controlcenter/PcApi.java`
> 类型：远端代理（→ ControlCenter 服务）
> 转发方式：V1 ApiServlet 前缀（`ApiUtil.doPress("ControlCenter", reqJson)`，action/op 反射路由）

## 方法列表

### 1. disList — 获取去重后的浏览器信息

```java
public List<PcInfoSource> disList(Integer eid, Integer projectid, Map<String, Object> conditionMap, Integer page, Integer pageSize)
```

**用途**：向 ControlCenter 查询项目可用 PC 浏览器列表，按 conditionMap 条件过滤并去重；PC 任务创建时用于浏览器选择。

**转发目标**：`action=pc, op=Pc.disList`（ControlCenter）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| eid | Integer | 是 | 企业 id，null 返回 null |
| projectid | Integer | 是 | 项目 id，null 返回 null |
| conditionMap | Map&lt;String,Object&gt; | 否 | 过滤条件（逐键并入 dataJson），null 视为空 |
| page | Integer | 否 | 页码 |
| pageSize | Integer | 否 | 每页大小 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | PcInfoSource | 浏览器信息对象（字段见 ControlCenter 服务 `Pc.disList`，代码未确认） |

**调用者**：
- `TaskServiceImpl.java`（`pcapi.disList(eid, projectid, conditionMap, 1, 1000)`）
- 注入于 `AbstractGenericServiceImpl`（`pcapi` 字段）

## 相关文档

- [00-分支索引](00-分支索引.md)
- [ControlCenter 服务](../../../设备控制中心（real-controlcenter）/07-开放接口文档/00-模块索引.md)
- [service-ClientApi](service-ClientApi.md)
