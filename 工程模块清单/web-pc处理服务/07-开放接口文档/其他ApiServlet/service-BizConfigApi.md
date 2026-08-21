# service-BizConfigApi — 业务编码配置查询

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/realcfg/BizConfigApi.java`（SpringHelper Bean `realcfg.BizConfigApi`）
> 类型：远端代理（→ RealCfg 服务）
> 转发方式：V1 ApiServlet 前缀（`ApiUtil.doPress("RealCfg", reqJson)`，action/op 反射路由）

## 方法列表

### 1. get — 查询单个业务编码配置

```java
public RealcfgBizConfig get(Integer bizCode) throws GeneralException
```

**用途**：按 bizCode 查询业务编码配置（任务创建/通知时取业务名称等信息）。

**转发目标**：`action=cfg, op=BizConfig.get`（RealCfg）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| bizCode | Integer | 是 | 业务编码，null 直接返回 null（debugLog ERROR） |

**返回参数**：`RealcfgBizConfig`（业务编码配置对象，字段见 RealCfg 服务 `BizConfig.get`，代码未确认）。

**调用者**：
- `TaskServiceImpl.java` /  / （`bizconfigapi.get(bizCode)`）
- `NoticeServiceImpl.java` / 

### 2. list — 查询业务编码列表

```java
public List<RealcfgBizConfig> list() throws GeneralException
```

**用途**：查询全部业务编码配置列表（空 data 请求体）。

**转发目标**：`action=cfg, op=BizConfig.list`（RealCfg）

**请求参数**：无（空 data 请求体）。

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | RealcfgBizConfig | 业务编码配置对象（字段见 RealCfg 服务 `BizConfig.list`，代码未确认） |

**调用者**：注入于 `AbstractGenericServiceImpl`（`bizconfigapi` 字段），供业务子类使用。

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealCfg 服务](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md)
- [service-McPcTaskApi](service-McPcTaskApi.md)
