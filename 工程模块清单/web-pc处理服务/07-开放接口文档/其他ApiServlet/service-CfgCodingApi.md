# service-CfgCodingApi — 错误码对照查询

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/realcfg/CfgCodingApi.java`（SpringHelper Bean `realcfg.CfgCodingApi`）
> 类型：远端代理（→ RealCfg 服务）
> 转发方式：V1 ApiServlet 前缀（`ApiUtil.doPress("RealCfg", reqJson)`，action/op 反射路由）

## 方法列表

### 1. list — 查询错误码对照信息

```java
public List<DbCoding> list() throws GeneralException
```

**用途**：拉取全部错误码对照表（DbCoding），用于报告中错误码到错误描述的映射。

**转发目标**：`action=cfg, op=CodingCfg.list`（RealCfg）

**请求参数**：无（空 data 请求体）。

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | DbCoding | 错误码对照对象（字段见 RealCfg 服务 `CodingCfg.list`，代码未确认） |

**调用者**：
- `CfgServiceImpl.java` / （`cfgcodingapi.list()`）
- 注入于 `AbstractGenericServiceImpl`（`cfgcodingapi` 字段）

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealCfg 服务](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md)
- [service-BizConfigApi](service-BizConfigApi.md)
