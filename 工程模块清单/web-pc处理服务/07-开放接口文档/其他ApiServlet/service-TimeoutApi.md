# service-TimeoutApi — 超时配置查询

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/realcfg/TimeoutApi.java`（SpringHelper Bean `realcfg.TimeoutApi`）
> 类型：远端代理（→ RealCfg 服务）
> 转发方式：get 走 V1 ApiServlet 前缀；getTimeoutConfig 走 HttpClient 直连 REST（`/v3/realcfg/project/getAdvancedConfig`）

## 方法列表

### 1. get — 获取项目组步骤级超时时间

```java
public TimeoutConfig get(Integer projectid, String taskid) throws GeneralException
```

**用途**：查询项目步骤级超时配置；taskid 以 `p` 开头时 businessType 置为 PC，否则为 WEB。

**转发目标**：`action=cfg, op=Timeout.get`（RealCfg，V1）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectid | Integer | 否 | 项目 id |
| taskid | String | 是 | 任务 id（`taskid.startsWith("p")` 区分 WEB/PC，null 会 NPE） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | TimeoutConfig | 超时配置对象（字段见 RealCfg 服务，代码未确认） |

**调用者**：`TaskServiceImpl.java`（`SpringHelper.getBean("realcfg.TimeoutApi")`）

### 2. getTimeoutConfig — 获取项目高级设置中的超时设置

```java
public RealcfgProAdvTimeoutConfig getTimeoutConfig(Integer projectId)
```

**用途**：查询项目高级设置 EXECUTE 类型的超时配置（app/web/pc 三端 ConfigDetail），通知里展示超时阈值。

**转发目标**：`GET {RealCfg}/v3/realcfg/project/getAdvancedConfig?project_id=X&type=EXECUTE`（HttpClient 直连，地址取自 `service.api.properties` 的 `RealCfg` 项）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectId | Integer | 否 | 项目 id（直接拼接到 query，无 null 校验） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (对象) | RealcfgProAdvTimeoutConfig | 高级超时配置对象（含 id/configType/projectId/content{app,web,pc}） |

**调用者**：
- `TaskServiceImpl.java`
- `NoticeServiceImpl.java` / 

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealCfg 服务](../../../平台配置（real-cfg）/07-开放接口文档/00-模块索引.md)
- [service-EnvConfigApi](service-EnvConfigApi.md)
