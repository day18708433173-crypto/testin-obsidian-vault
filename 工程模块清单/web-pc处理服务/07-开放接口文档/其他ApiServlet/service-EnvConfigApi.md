# service-EnvConfigApi — 环境配置查询

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/realcfg/EnvConfigApi.java`（SpringHelper Bean `realcfg.EnvConfigApi`）
> 类型：远端代理（→ RealCfg 服务）
> 转发方式：V1 ApiServlet 前缀（`ApiUtil.doPress("RealCfg", reqJson)`，action/op 反射路由）

## 方法列表

### 1. get — 获取环境配置信息

```java
public RealCfgEnvConfig get(Integer envId, Integer eid) throws GeneralException
```

**用途**：按 envId + eid 查询环境配置（精准测试等场景取环境参数，判断是否开启 CICC 覆盖率采集）。

**转发目标**：`action=cfg, op=EnvCfg.get`（RealCfg）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| envId | Integer | 是 | 环境 id，null 抛 `paraInvalid` |
| eid | Integer | 是 | 企业 id，null 抛 `paraInvalid` |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | RealCfgEnvConfig | 环境配置对象（字段见 RealCfg 服务 `EnvCfg.get`，代码未确认） |

**调用者**：
- `TaskServiceImpl.java`
- `TaskProcessServiceImpl.java` / （精准测试流程）
- `ReportServiceImpl.java` / 
- `NoticeServiceImpl.java`

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealCfg 服务](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md)
- [service-DBConfigApi](service-DBConfigApi.md)、[service-PreciseTestApi](service-PreciseTestApi.md)
