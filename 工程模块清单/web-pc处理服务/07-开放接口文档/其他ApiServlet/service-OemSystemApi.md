# service-OemSystemApi — OEM 系统参数查询

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/oemparam/OemSystemApi.java`
> 类型：远端代理（→ UserManager 服务）
> 转发方式：V1 ApiServlet 前缀（`ApiUtil.doPress("UserManager", reqJson)`，action/op 反射路由）

## 方法列表

### 1. getSystemGroup — 查询配置类型列表

```java
public static List<SystemParam> getSystemGroup(String param_group) throws GeneralException
```

**用途**：按参数组名查询 OEM 系统参数（如 logo、名称、域名等 OEM 定制配置），Config 初始化与通知模板取 OEM 参数时使用。静态方法。

**转发目标**：`action=user, op=SystemParam.list`（UserManager）

**请求参数**：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| param_group | String | 是 | 配置类型/参数组名 |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | SystemParam | 系统参数对象（字段见 UserManager 服务 `SystemParam.list`，代码未确认） |

**调用者**：
- `utils/Config.java` / （系统启动加载 OEM 参数）
- `NoticeServiceImpl.java` / （通知内容 OEM 化）

## 相关文档

- [00-分支索引](00-分支索引.md)
- [UserManager 服务](../../../平台基础功能服务/00-首页.md)
- [service-ReportApi](service-ReportApi.md)
