# service-DeviceV3Api — 设备信息查询代理（V3）

> 分支：syy.release.z7.8.1.0
> 源文件：`src/main/java/cn/testin/realweb/api/device/DeviceV3Api.java`（@Component）
> 类型：远端代理（→ RealScheduling / ControlCenter 服务）
> 转发方式：V3 REST，经 `ServiceRemoteV3Api.remotePost`；域名用 `ApiUtil.getServiceApiUrl(...)` 按服务名解析

## 方法列表

### 1. getDeviceInfo — 查询任务设备信息（→ RealScheduling）

```java
public List<DeviceInfoResponseDTO> getDeviceInfo(List<String> deviceIds, Integer taskType, Integer projectId, Integer eid)
```

**用途**：按设备 id 列表 + 任务类型查询任务维度设备信息；deviceIds 或 taskType 为空时返回空列表。

**转发目标**：

```java
String domain = ApiUtil.getServiceApiUrl("RealScheduling");
String url = domain + "/v3/device_task/device_task/get_device_task_info"; // POST
```

**请求参数**（body `DeviceQueryRequestDTO`）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| deviceIds | List&lt;String&gt; | 是 | 设备 id 列表，空返回空列表 |
| taskType | Integer | 是 | 任务类型，null 返回空列表 |
| projectId | Integer | 否 | 项目 id |
| eid | Integer | 否 | 企业 id |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | DeviceInfoResponseDTO | 任务设备信息对象（字段见 RealScheduling 服务，代码未确认） |

**调用者**：
- `WebQuartz.java` — taskType=WEB
- `McPcQuartz.java` — taskType=PC

### 2. getDeviceInfo — 查询设备列表（→ ControlCenter）

```java
public List<DeviceInfoResponse> getDeviceInfo(DeviceQueryRequestDTO request) throws GeneralException
```

**转发目标**：

```java
String domain = ApiUtil.getServiceApiUrl("ControlCenter");
String url = domain + "/v3/ControlCenter/device/device_list"; // POST
```

**请求参数**（body `DeviceQueryRequestDTO`，无 null 校验）：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| request | DeviceQueryRequestDTO | 否 | 设备查询条件对象（deviceIds 等，字段见 ControlCenter 服务） |

**返回参数**：

| 字段 | 类型 | 说明 |
|------|------|------|
| (列表元素) | DeviceInfoResponse | 设备信息对象（字段见 ControlCenter 服务，代码未确认） |

**调用者**：`McPcQuartz.java` — PC 设备状态查询。

## 相关文档

- [00-分支索引](00-分支索引.md)
- [RealScheduling](../../../任务调度服务（real-scheduling）/07-开放接口文档/00-模块索引.md)、[ControlCenter](../../../设备控制中心（real-controlcenter）/07-开放接口文档/00-模块索引.md)
- [WebQuartz](WebQuartz.md) / [McPcQuartz](McPcQuartz.md)
